# Testy jednostkowe

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Strategia testowania](./01-strategia-testowania.md), [Funkcje](../04-javascript/02-funkcje.md), [Asynchroniczność](../04-javascript/08-asynchronicznosc.md)

Test jednostkowy sprawdza pojedynczą jednostkę logiki w izolacji — szybko i precyzyjnie. Świetny dla **logiki domenowej** (obliczenia, walidacja, formatowanie, reducery, utils). Ten plik to warsztat: anatomia testu, dublery testowe i praktyki, które odróżniają testy pomocne od kruchych.

## Anatomia testu: AAA

```js
import { describe, it, expect } from 'vitest';

describe('calculateDiscount', () => {
  it('applies percentage discount above threshold', () => {
    // ARRANGE — przygotuj dane/kontekst
    const cart = { total: 200, items: 3 };
    // ACT — wykonaj testowaną jednostkę
    const result = calculateDiscount(cart, { type: 'percent', value: 10, min: 100 });
    // ASSERT — sprawdź wynik
    expect(result).toBe(20);
  });
});
```

- **Arrange-Act-Assert** — trzy wyraźne fazy; jeden logiczny warunek na test.
- **Nazwa opisuje zachowanie** („zwraca 0 poniżej progu"), nie metodę („test calculateDiscount 2").
- Test ma być **oczywisty** — czytający rozumie intencję bez debugera; unikaj logiki (pętli, warunków) w teście.

## Matchery i asercje

```js
expect(x).toBe(5);                 // === (prymitywy, referencje)
expect(obj).toEqual({ a: 1 });     // głębokie porównanie wartości
expect(arr).toContain(3);
expect(fn).toThrow('Invalid');     // funkcja rzuca (przekaż referencję, nie wynik!)
expect(mock).toHaveBeenCalledWith(42);
expect(value).toMatchObject({ id: 1 });   // podzbiór pól
```

`toBe` vs `toEqual` to klasyczna pułapka (referencja vs zawartość — [typy](../04-javascript/01-typy-i-zmienne.md)).

## Testy asynchroniczne

```js
it('fetches user', async () => {
  await expect(getUser(1)).resolves.toEqual({ id: 1 });
  await expect(getUser(-1)).rejects.toThrow(NotFoundError);
});
```

Zawsze `await`/zwróć Promise — inaczej test „przechodzi" przed wykonaniem asercji (fałszywy zielony). Kontrola czasu: **fake timers** (`vi.useFakeTimers()` + `vi.advanceTimersByTime()`) do testowania [debounce/throttle](../09-wydajnosc/03-optymalizacja-dzialania.md), setTimeout, pollingu — bez realnego czekania.

## Test doubles: mock, stub, spy, fake

Rozróżnienie, o które pytają:

- **Stub** — dostarcza gotowe odpowiedzi (zwraca ustaloną wartość zamiast prawdziwej implementacji).
- **Spy** — obserwuje wywołania (czy/ile razy/z czym), zwykle bez zmiany zachowania.
- **Mock** — stub + spy + wbudowane oczekiwania weryfikowane w teście.
- **Fake** — działająca lekka implementacja (np. in-memory repo zamiast bazy).

```js
const save = vi.fn().mockResolvedValue({ id: 1 });   // mock funkcji
vi.spyOn(analytics, 'track');                         // spy na istniejącej metodzie
vi.mock('./api', () => ({ getUser: vi.fn() }));       // mock modułu
```

**Zasada mockowania:** mockuj **granice** (sieć, czas, losowość, storage, moduły zewnętrzne), nie wnętrze testowanej jednostki. Nadmiar mocków = test sprawdza mocki, nie kod; sygnał, że test jest za nisko/jednostka za bardzo sprzężona. Mock sieci najlepiej na poziomie HTTP (**MSW**), nie przez podmianę `fetch` — [komponenty](./03-testy-komponentow-i-integracyjne.md).

## TDD (Test-Driven Development)

Cykl **Red → Green → Refactor**: napisz padający test → minimalny kod, by przeszedł → posprzątaj. Wartość: projektuje API „od strony użycia", daje siatkę bezpieczeństwa do refaktoru, wymusza testowalność. Nie dogmat — świetny dla logiki o jasnych regułach (algorytmy, parsery, kalkulacje), mniej naturalny dla eksploracyjnego UI. Umieć wyjaśnić cykl i kiedy go stosujesz.

## Cechy dobrego unit testu (FIRST)

- **Fast** — milisekundy; wolne testy się pomija.
- **Isolated** — niezależny od innych i od kolejności; własny setup/teardown, brak współdzielonego mutowalnego stanu.
- **Repeatable** — deterministyczny: żadnej realnej sieci, losowości, `Date.now()` bez kontroli (wstrzykuj zegar/fake timers).
- **Self-validating** — jasny pass/fail, bez ręcznej interpretacji.
- **Timely** — pisany blisko kodu (najlepiej przed/w trakcie).

## Pułapki i częste błędy

- `toBe` na obiektach zamiast `toEqual`.
- Test async bez `await` — fałszywy zielony.
- Over-mocking — test odzwierciedla implementację, pęka przy refaktorze i niczego nie gwarantuje.
- Współdzielony stan między testami (brak resetu mocków: `vi.clearAllMocks()` w `beforeEach`).
- Testowanie prywatnych detali zamiast publicznego zachowania jednostki.
- Niedeterminizm: realny czas/losowość/`Math.random`/`Date` bez wstrzyknięcia — flaky.
- Jeden test asertujący dziesięć niepowiązanych rzeczy — po awarii nie wiadomo, co pękło.

## Pytania rekrutacyjne

1. **Co to AAA i po co?** — struktura testu; czytelność, jeden warunek.
2. **`toBe` vs `toEqual`?** — referencja vs głębokie porównanie.
3. **Mock vs stub vs spy vs fake?** — definicje + kiedy które; co mockować (granice).
4. **Jak testujesz kod zależny od czasu?** — fake timers; determinizm.
5. **Wyjaśnij cykl TDD i kiedy go stosujesz.** — red/green/refactor; logika domenowa vs eksploracyjne UI.
6. **Dlaczego over-mocking to problem?** — test implementacji; kruchość; brak realnej pewności.

## Dalsza lektura

- [Vitest — dokumentacja](https://vitest.dev/)
- [Martin Fowler: Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html) — klasyk rozróżnienia dublerów
- [Testing Library: guiding principles](https://testing-library.com/docs/guiding-principles/)
