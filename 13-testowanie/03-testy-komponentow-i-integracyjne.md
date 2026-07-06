# Testy komponentów i integracyjne

> **Poziom:** 🟡 średni → 🔴 (praktyka user-centric)
> **Wymagana wiedza:** [Strategia testowania](./01-strategia-testowania.md), [Testy jednostkowe](./02-testy-jednostkowe.md), [Model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)

To najwartościowsze testy we frontendzie ([trofeum](./01-strategia-testowania.md)): sprawdzają komponenty i ich współpracę **tak, jak używa ich użytkownik** — klika, pisze, widzi. Filozofia jest jedna, niezależnie od frameworka: **testuj zachowanie widoczne dla użytkownika, nie wewnętrzną implementację**.

## Zasada przewodnia Testing Library

> „Im bardziej testy przypominają sposób użycia oprogramowania, tym większą dają pewność."

Konsekwencje praktyczne:
- Znajduj elementy tak, jak **użytkownik/technologia asystująca** — po **roli i dostępnej nazwie**, tekście, labelu; nie po klasach CSS czy strukturze DOM.
- Wchodź w interakcje realistycznie (`userEvent` symuluje pełne zdarzenia: focus, keydown, input — bliżej prawdy niż surowy `fireEvent`).
- Asertuj **efekt widoczny** (pojawił się komunikat, zniknął spinner), nie stan wewnętrzny komponentu.

```js
import { render, screen } from '@testing-library/react';   // analogicznie Vue/Svelte/Angular
import userEvent from '@testing-library/user-event';

it('pokazuje błąd dla pustego e-maila', async () => {
  render(<LoginForm />);
  await userEvent.click(screen.getByRole('button', { name: /zaloguj/i }));
  expect(await screen.findByText(/podaj e-mail/i)).toBeInTheDocument();
});
```

**Selektory wg priorytetu (od najlepszego):** `getByRole` (z `name`) → `getByLabelText` (formularze) → `getByText` → `getByPlaceholderText` → **ostateczność:** `getByTestId` (`data-testid`). Preferencja ról **wymusza dostępny HTML** — testy i [a11y](../10-dostepnosc/02-aria-i-semantyka.md) napędzają się nawzajem (jeśli nie umiesz znaleźć przycisku po roli, czytnik ekranu też go nie znajdzie).

## Warianty zapytań: get / query / find

- `getBy*` — rzuca, gdy nie znajdzie (element **musi** być). 
- `queryBy*` — zwraca `null` (do asercji **braku**: `expect(queryByText(...)).not.toBeInTheDocument()`).
- `findBy*` — async, czeka aż element się pojawi (dane, animacje) — zamiast ręcznych `sleep`.

## Testy integracyjne — komponent + dane + interakcja

Prawdziwa wartość: przetestuj kawałek aplikacji z realną współpracą części, mockując tylko **granicę sieci**:

```js
// MSW — mock na poziomie HTTP: komponent robi prawdziwy fetch, MSW odpowiada
server.use(http.get('/api/products', () => HttpResponse.json([{ id: 1, name: 'Buty' }])));

it('ładuje i wyświetla produkty', async () => {
  render(<ProductList />);
  expect(screen.getByText(/ładowanie/i)).toBeInTheDocument();       // stan loading
  expect(await screen.findByText('Buty')).toBeInTheDocument();       // po danych
});
```

- **MSW (Mock Service Worker)** przechwytuje requesty na poziomie sieci → komponent + hook + [warstwa API](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md) działają realnie; jeden zestaw handlerów służy testom **i** dev-serverowi. Lepsze niż mockowanie `fetch`/modułu (testujesz integrację, nie atrapę).
- Testuj **stany asynchroniczne** ([wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)): loading, sukces, **błąd** (`server.use` z 500), **empty**, race — najczęściej pomijane, a najczęściej psute.
- Renderuj z realnymi providerami (router, store, query-client) — to właśnie „integracja".

## Co i jak asertować

- ✅ „po kliknięciu pojawia się potwierdzenie", „lista pokazuje 3 pozycje", „błąd API pokazuje komunikat i przycisk ponów".
- ❌ „stan `isOpen` === true", „metoda `handleClick` została wywołana", „komponent ma klasę `.active`" — to implementacja.
- Migawki (snapshots) — oszczędnie: wielkie snapshoty DOM są kruche i nikt ich nie czyta przy zmianie; lepsze celowane asercje. Ewentualnie małe, inline snapshots dla ustrukturyzowanych danych.

## Pułapki i częste błędy

- Selektory po klasach/strukturze DOM (`container.querySelector('.btn-primary')`) — pękają przy każdej zmianie stylu/struktury.
- `fireEvent` tam, gdzie `userEvent` jest realistyczniejszy (pomija focus/klawiaturę).
- Brak `await` przy `findBy`/`userEvent` (są async) → „act warnings", flaky.
- Testowanie stanu wewnętrznego zamiast wyrenderowanego efektu.
- Mockowanie wszystkiego (w tym własnych komponentów-dzieci) → test niczego realnego nie sprawdza.
- Pomijanie ścieżek błędu/empty — testujesz tylko happy path.
- Gigantyczne snapshoty aktualizowane w ciemno (`--updateSnapshot` bez czytania diffa).

## Pytania rekrutacyjne

1. **Jaka jest główna zasada Testing Library?** — testuj jak użytkownik; selektory po roli/nazwie.
2. **get vs query vs find?** — obecność wymagana / brak / async oczekiwanie.
3. **Jak mockujesz API w testach komponentów i czemu MSW?** — poziom sieci, jeden mock dla testów i dev, realna integracja.
4. **Czemu selektory po roli są lepsze niż po klasach?** — odporność na refaktor + wymuszona dostępność.
5. **Jak testujesz stany ładowania/błędu?** — findBy dla async, MSW z 500/empty, asercje widocznych komunikatów.
6. **Kiedy snapshoty pomagają, a kiedy szkodzą?** — małe/ustrukturyzowane vs wielkie kruche DOM-y.

## Dalsza lektura

- [Testing Library — docs + Guiding Principles](https://testing-library.com/docs/)
- [MSW (Mock Service Worker)](https://mswjs.io/)
- [Kent C. Dodds: Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
