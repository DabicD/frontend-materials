# Strategia testowania

> **Poziom:** 🟡 średni → 🔴 (decyzje)
> **Wymagana wiedza:** podstawy [JS](../04-javascript/02-funkcje.md)

Testy istnieją po to, byś mógł **zmieniać kod bez strachu** i dostarczać szybciej — nie dla samego „pokrycia". Strategia testowania to odpowiedź na pytanie: *co, na jakim poziomie i po co* testujemy, by uzyskać maksimum pewności przy rozsądnym koszcie. To pytanie architektoniczne, nie tylko techniczne.

## Rodzaje testów

- **Jednostkowe (unit)** — pojedyncza funkcja/moduł w izolacji; szybkie, precyzyjne ([szczegóły](./02-testy-jednostkowe.md)).
- **Integracyjne** — współpraca kilku części (komponent + hook + warstwa API na mocku); najlepszy stosunek pewności do kosztu we frontendzie ([komponenty i integracja](./03-testy-komponentow-i-integracyjne.md)).
- **E2E (end-to-end)** — cały flow w prawdziwej przeglądarce (logowanie → zakup); najwyższa pewność, najwyższy koszt ([e2e i specjalistyczne](./04-testy-e2e-i-specjalistyczne.md)).
- **Statyczne** — [TypeScript](../06-typescript/01-podstawy-typowania.md), [ESLint](../14-narzedzia/04-jakosc-kodu.md): „test", który łapie całą klasę błędów za darmo, zanim odpalisz cokolwiek.

## Piramida vs trofeum

**Piramida testów** (klasyka): dużo unit, mniej integracyjnych, garść e2e. Sensowna, ale w interaktywnym frontendzie unit-testy izolowanych kawałków UI dają mniej pewności niż koszt sugeruje (dużo mockowania, testowanie implementacji).

**Testing Trophy** (dla frontendu, Kent C. Dodds): fundament to **statyka** (TS+lint), największy nacisk na **integracyjne**, mniej unit, garść e2e. Motto: **„pisz testy dające najwięcej pewności"** — a to zwykle testy integracyjne komponentów (blisko tego, jak używa ich użytkownik), nie unit każdej funkcji.

```
        /\        e2e (mało, krytyczne ścieżki)
       /  \
      /____\      integracyjne  ← największa wartość we froncie
     /______\     unit (logika, edge case'y)
    /________\    statyka (TS + lint) — fundament
```

Nie traktuj kształtu dogmatycznie: bogata logika domenowa (kalkulacje, parsery) → więcej unit; aplikacja głównie „UI + API" → więcej integracyjnych.

## Co testować (i czego nie)

**Testuj:**
- logikę domenową i edge case'y (obliczenia, walidacja, formatowanie, reducery),
- **zachowanie z perspektywy użytkownika** („klika kup → widzi potwierdzenie"),
- kontrakty (warstwa API, publiczne API modułów — [struktura](../08-architektura-aplikacji/04-struktura-projektu.md)),
- rzeczy, które **się psuły** (regression test do każdego buga — najtańsza pewność),
- krytyczne ścieżki biznesowe (rejestracja, płatność) — e2e.

**Nie testuj:**
- szczegółów implementacji (nazwy metod, stan wewnętrzny, kolejność wywołań) — testy pękają przy refaktorze, choć zachowanie się nie zmieniło,
- bibliotek third-party (framework działa),
- trywialnych getterów/stałych,
- „na 100% pokrycia" — pokrycie to narzędzie diagnostyczne (gdzie są dziury), nie cel; 100% często testuje bzdury.

## Zasady dobrej strategii

1. **Pewność, nie pokrycie** — pytaj „czy śpię spokojnie przy deployu?", nie „ile procent?".
2. **Testuj zachowanie, nie implementację** — to jedyna droga do testów, które przeżywają refaktor (leitmotiv całej sekcji).
3. **Szybka pętla zwrotna** — unit/integracyjne na każdym zapisie (watch), e2e na [CI](../16-devops-dla-frontendu/01-ci-cd.md); wolne testy = pomijane testy.
4. **Deterministyczność** — testy losowo padające (flaky) niszczą zaufanie bardziej niż brak testów; eliminuj bezwzględnie ([e2e](./04-testy-e2e-i-specjalistyczne.md)).
5. **Testy to kod** — czytelność, brak duplikacji, utrzymywalność; zły test to dług, nie aktywo.

## Ekosystem (świadomość)

- **Test runner + asercje:** Vitest (nowoczesny, szybki, natywny ESM/TS — de facto standard w Vite) lub Jest.
- **Komponenty:** Testing Library (podejście user-centric — [komponenty](./03-testy-komponentow-i-integracyjne.md)).
- **E2E:** Playwright (dziś dominujący) / Cypress.
- **Mock sieci:** MSW (Mock Service Worker) — przechwytuje na poziomie sieci, jeden mock dla testów i dev.

## Pułapki i częste błędy

- Cel „X% coverage" jako KPI → testy-atrapy podbijające metrykę bez pewności.
- Testowanie implementacji → czerwony pakiet po każdym refaktorze mimo działającego produktu.
- Piramida na siłę tam, gdzie trofeum pasuje lepiej (dużo kruchych unit-testów UI).
- Ignorowane flaky testy („odpal jeszcze raz") → erozja zaufania do całego pakietu.
- Brak testów regresyjnych do bugów → te same błędy wracają.
- Traktowanie TS/lint jako „nie testów" — to najtańsza warstwa pewności.

## Pytania rekrutacyjne

1. **Po co piszemy testy?** — pewność przy zmianach i szybkość dostarczania; nie pokrycie.
2. **Piramida vs trofeum — co i czemu we froncie?** — nacisk na integracyjne; rola statyki; kiedy więcej unit.
3. **Czego NIE testować?** — implementacji, bibliotek, trywialności; pokrycie jako diagnostyka.
4. **Co znaczy „testuj zachowanie, nie implementację"?** — testy odporne na refaktor; perspektywa użytkownika.
5. **Jak podejdziesz do testów w projekcie bez testów?** — statyka najpierw, potem integracyjne krytycznych ścieżek + regresje do bugów.

## Dalsza lektura

- [Kent C. Dodds: The Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) i [Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests)
- [Martin Fowler: Testing strategies / TestPyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
