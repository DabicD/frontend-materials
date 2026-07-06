# Testy E2E i specjalistyczne

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Strategia testowania](./01-strategia-testowania.md), [Testy komponentów](./03-testy-komponentow-i-integracyjne.md)

Testy E2E i specjalistyczne stoją na szczycie [trofeum](./01-strategia-testowania.md): najwyższa pewność, najwyższy koszt i największa podatność na flakiness. Sekret dobrego E2E: **mało, ale niezawodnych** — pokrywających krytyczne ścieżki biznesowe, nie wszystko.

## E2E — cały flow w prawdziwej przeglądarce

Testują aplikację od strony użytkownika end-to-end: prawdziwa przeglądarka, prawdziwa nawigacja, (zwykle) prawdziwy lub bliski produkcji backend.

```js
import { test, expect } from '@playwright/test';

test('użytkownik kupuje produkt', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('link', { name: 'Buty trailowe' }).click();
  await page.getByRole('button', { name: 'Dodaj do koszyka' }).click();
  await page.getByRole('link', { name: /koszyk/i }).click();
  await expect(page.getByText('1 produkt')).toBeVisible();
});
```

- **Playwright** (dziś dominujący: multi-browser Chromium/Firefox/WebKit, auto-waiting, trace viewer, równoległość) / **Cypress** (świetny DX, głównie Chromium).
- Te same **selektory po roli/nazwie** co w [komponentach](./03-testy-komponentow-i-integracyjne.md) — spójność i wymuszona [dostępność](../10-dostepnosc/02-aria-i-semantyka.md).

**Walka z flakiness (kluczowa umiejętność):**
- **Auto-waiting/web-first assertions** (`toBeVisible`) zamiast sztywnych `waitForTimeout` — czekaj na warunek, nie na czas.
- **Stabilne dane** — seed/reset stanu przed testem; nie polegaj na danych produkcyjnych.
- **Izolacja** — każdy test niezależny, własny użytkownik/koszyk; brak zależności od kolejności.
- **Mock zewnętrznych, niekontrolowanych usług** (płatności, third-party) — [MSW](./03-testy-komponentow-i-integracyjne.md)/route interception; realny własny backend.
- Retry na CI + **trace/wideo/screenshot** przy porażce (Playwright zapisuje) — debug bez reprodukcji lokalnie.

**Zakres:** tylko **krytyczne ścieżki** (logowanie, rejestracja, checkout, główny happy path + kluczowe błędy). E2E na każdy edge case = wolny, kruchy pakiet, który zespół zacznie ignorować.

## Testy wizualne (visual regression)

Wykrywają **niezamierzone zmiany wyglądu** przez porównanie zrzutów (pixel/AI diff) z baseline:

- Narzędzia: Playwright `toHaveScreenshot`, Chromatic (dla Storybook), Percy, Lost Pixel.
- Łapią to, czego asercje nie widzą: rozjechany layout, zły kolor, regresja po zmianie CSS/[refaktorze designu](../03-css/09-architektura-css.md).
- Wyzwania: **niestabilność** (fonty, animacje, daty, dane losowe → maskuj/zamrażaj), zarządzanie baseline (świadoma akceptacja zmian), koszt.
- Najlepiej na **komponentach w Storybook** (izolacja, warianty, stany) niż na całych stronach.

## Testy dostępności (automatyczne)

Integracja **axe-core** z pipeline ([a11y w praktyce](../10-dostepnosc/03-a11y-w-praktyce.md)):

```js
// w komponentach
expect(await axe(container)).toHaveNoViolations();
// w e2e
const results = await new AxeBuilder({ page }).analyze();
expect(results.violations).toEqual([]);
```

Łapią ~30–40% problemów (kontrast, brakujące nazwy, role) — reszta wymaga [testów manualnych](../10-dostepnosc/03-a11y-w-praktyce.md). Tanie, warto w każdym projekcie.

## Contract testing

Gwarantuje zgodność kontraktu między frontendem (consumer) a API (provider) **bez** pełnych testów integracyjnych obu naraz:

- **Pact** (consumer-driven): frontend definiuje oczekiwania → generuje kontrakt → provider weryfikuje, że go spełnia.
- Alternatywa: walidacja mocków/typów względem [OpenAPI](../12-komunikacja-z-api/01-rest.md)/[schemy GraphQL](../12-komunikacja-z-api/02-graphql.md) (codegen + typy — [TS w praktyce](../06-typescript/05-typescript-w-praktyce.md)).
- Rozwiązuje realny ból: mock w testach mówi „X", a API zmieniło się na „Y" — testy zielone, produkcja pada. Szczególnie ważne w [mikrofrontendach](../08-architektura-aplikacji/03-microfrontends-i-monorepo.md)/wielu zespołach.

## Testy wydajności i inne

- **Lighthouse CI** — budżety [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md) na PR ([CI/CD](../16-devops-dla-frontendu/01-ci-cd.md)); regresja wagi bundle'a/metryk blokuje merge.
- **Property-based testing** (fast-check) — generuje setki losowych wejść przeciw właściwościom funkcji (np. „`parse(format(x)) === x`") — świetne dla logiki, łapie edge case'y, o których nie pomyślisz.
- **Mutation testing** (Stryker) — ocenia **jakość** testów: wprowadza mutacje kodu i sprawdza, czy testy je wykryją (pokrycie mówi, że linia się wykonała; mutation — czy jest naprawdę sprawdzona).

## Pułapki i częste błędy

- Za dużo E2E (testowanie edge case'ów, które należą do integracyjnych/unit) → wolno i flaky.
- `waitForTimeout(3000)` zamiast web-first assertions — źródło flakiness nr 1.
- Zależność testów od siebie/kolejności/współdzielonych danych.
- Ignorowanie flaky E2E („retry załatwia") zamiast naprawy przyczyny → erozja zaufania.
- Visual regression bez maskowania dynamiki → ciągłe fałszywe alarmy → wyłączenie.
- Mock w testach rozjechany z realnym API (brak contract testing) → zielono lokalnie, czerwono na produkcji.
- E2E na produkcji z realnymi płatnościami/mailami — środowiska testowe i mocki granic zewnętrznych.

## Pytania rekrutacyjne

1. **Ile i jakich testów E2E?** — mało, krytyczne ścieżki; reszta niżej w trofeum.
2. **Jak walczysz z flaky E2E?** — auto-wait/web-first assertions, izolacja, seed danych, mock granic, trace.
3. **Co daje visual regression i jakie ma problemy?** — wyłapanie zmian wyglądu; niestabilność, baseline.
4. **Po co contract testing?** — zgodność front↔API bez pełnej integracji; rozjazd mocka z realnym API.
5. **Pokrycie vs mutation testing?** — „linia wykonana" vs „linia naprawdę sprawdzona".
6. **Jak pilnujesz wydajności w CI?** — Lighthouse CI + budżety.

## Dalsza lektura

- [Playwright — docs + best practices](https://playwright.dev/docs/best-practices)
- [Pact — contract testing](https://docs.pact.io/)
- [Stryker Mutator](https://stryker-mutator.io/) / [fast-check](https://fast-check.dev/)
