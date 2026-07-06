# Frontend system design

> **Poziom:** 🔴 zaawansowany (senior i wyżej)
> **Wymagana wiedza:** większość skarbnicy — to synteza

Na poziomie senior rozmowa techniczna często obejmuje **system design**: „zaprojektuj autocomplete / news feed / Google Sheets". To nie kodowanie, lecz **projektowanie na wysokim poziomie** — sprawdza, czy umiesz łączyć wiedzę z całej skarbnicy w spójny system, rozważać kompromisy i komunikować decyzje. Backendowy system design (bazy, sharding) ma tu frontendowy odpowiednik: komponenty, przepływ danych, wydajność, stany.

## Framework RADIO (struktura odpowiedzi)

Popularna metoda porządkująca rozmowę — trzymaj się jej, by nie zgubić wątku:

- **R — Requirements** — doprecyzuj zakres: funkcje, użytkownicy, urządzenia, skala, ograniczenia. **Zawsze zaczynaj tutaj** (jak w [zadaniach praktycznych](./03-zadania-praktyczne.md)) — projektowanie bez wymagań to czerwona flaga.
- **A — Architecture/high-level** — komponenty i moduły, ich odpowiedzialności, przepływ danych między nimi ([model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md), [struktura](../08-architektura-aplikacji/04-struktura-projektu.md)).
- **D — Data model** — jakie dane, kształt, skąd, gdzie żyją (server vs client state — [zarządzanie stanem](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)).
- **I — Interfaces/API** — kontrakty: API komponentów (props/events) i sieciowe ([REST](../12-komunikacja-z-api/01-rest.md)/[GraphQL](../12-komunikacja-z-api/02-graphql.md), [real-time](../12-komunikacja-z-api/03-czas-rzeczywisty.md)).
- **O — Optimizations** — wydajność, dostępność, UX brzegowy, skalowanie.

## Wymiary, o których trzeba pomyśleć (checklista seniora)

Dobra odpowiedź dotyka wielu warstw — to test, czy widzisz frontend całościowo:

- **Wydajność:** [ładowanie](../09-wydajnosc/02-optymalizacja-ladowania.md) (splitting, lazy), [runtime](../09-wydajnosc/03-optymalizacja-dzialania.md) (wirtualizacja, debounce), [CWV](../09-wydajnosc/01-core-web-vitals.md), [renderowanie](../08-architektura-aplikacji/01-wzorce-renderowania.md) (CSR/SSR).
- **Dane:** [fetching, cache, stany, race conditions, optimistic](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md), [warstwy cache](../09-wydajnosc/04-strategie-cachowania.md).
- **Stan:** [taksonomia](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md) — co lokalne, co globalne, co w URL, co server state.
- **Dostępność:** [klawiatura, ARIA, focus](../10-dostepnosc/03-a11y-w-praktyce.md) — senior nie pomija a11y.
- **UX stanów:** loading/error/empty, optymistyczność, offline.
- **Bezpieczeństwo:** [XSS przy user-content](../11-bezpieczenstwo/01-ataki-na-frontend.md), [auth](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md).
- **Skalowalność kodu:** [struktura/granice](../08-architektura-aplikacji/04-struktura-projektu.md), reużywalność.
- **Edge cases:** wolna sieć, duże dane, błędy API, konkurencyjne akcje, i18n.

Nie wszystko naraz — **komunikuj priorytety** („skupię się na przepływie danych i wydajności listy, bo to sedno") i pytaj rozmówcę, co go najbardziej interesuje.

## Przykład 1: Autocomplete / typeahead

- **R:** źródło (lokalne/API), min. znaków, ile wyników, klawiatura, mobile, skala (rozmiar słownika)?
- **A:** input → kontroler zapytań → cache → lista wyników; komponent [combobox (ARIA)](../10-dostepnosc/02-aria-i-semantyka.md).
- **D/I:** endpoint `GET /search?q=`, kształt wyników, [paginacja/limit](../12-komunikacja-z-api/01-rest.md).
- **O — sedno tego zadania:**
  - **[Debounce](../09-wydajnosc/03-optymalizacja-dzialania.md)** wejścia (nie request na każdą literę).
  - **[Anulowanie + race conditions](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)** — stary wynik nie może nadpisać nowego ([AbortController](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).
  - **[Cache](../09-wydajnosc/04-strategie-cachowania.md)** zapytań (te same litery → bez requestu).
  - **[Dostępność](../10-dostepnosc/02-aria-i-semantyka.md):** role combobox/listbox, `aria-activedescendant`, strzałki/Enter/Esc.
  - Stany: ładowanie, brak wyników, błąd; podświetlanie dopasowań; mobile.

## Przykład 2: Infinite scroll news feed

- **R:** rodzaj treści, media, real-time?, skala (nieskończona lista).
- **A/O:**
  - **[Wirtualizacja](../09-wydajnosc/03-optymalizacja-dzialania.md)** — nie trzymaj 10 000 postów w DOM.
  - **[IntersectionObserver](../05-dom-i-web-api/05-observery.md)** sentinel do doładowania (nie scroll listener).
  - **[Cursor pagination](../12-komunikacja-z-api/01-rest.md)** (stabilna dla zmieniających się danych).
  - **[Optymistyczne akcje](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)** (like), [cache i inwalidacja](../09-wydajnosc/04-strategie-cachowania.md).
  - Media: [lazy loading, wymiary vs CLS](../02-html/04-multimedia-i-grafika.md), [priorytety](../09-wydajnosc/02-optymalizacja-ladowania.md).
  - Zachowanie pozycji scrolla przy nawigacji wstecz ([routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)); real-time nowych postów ([SSE/WebSocket](../12-komunikacja-z-api/03-czas-rzeczywisty.md)).

## Przykład 3: Kolaboracyjny edytor (np. arkusz/dokument)

- Sedno: **[real-time](../12-komunikacja-z-api/03-czas-rzeczywisty.md)** (WebSocket), rozwiązywanie konfliktów (CRDT/OT — koncepcyjnie), [optymistyczne update'y](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md), [wirtualizacja](../09-wydajnosc/03-optymalizacja-dzialania.md) wielkiej siatki, [Web Workers](../05-dom-i-web-api/06-web-workers.md) do ciężkich obliczeń, offline + resync, presence (kursory innych).

## Jak oceniają (i jak wypaść dobrze)

- **Proces i komunikacja** > „jedna słuszna odpowiedź" — system design jest otwarty, prowadź dialog.
- **Trade-offy** — każda decyzja ma koszt; wypowiadaj je („SSR poprawi LCP, ale doda złożoność i koszt serwera").
- **Głębia na żądanie** — rozmówca drąży wybrany obszar; miej „drugie dno" (np. jak dokładnie działa Twój cache).
- **Realizm** — nie projektuj Figmy dla licznika; skaluj rozwiązanie do wymagań ([YAGNI](../15-inzynieria-oprogramowania/01-czysty-kod.md)).
- **Przyznawaj granice** — „tu wszedłbym głębiej z zespołem/researchem" jest OK.

## Pułapki i częste błędy

- Skok do rozwiązania bez zebrania wymagań (R w RADIO).
- Skupienie tylko na UI, pominięcie danych/stanu/wydajności/a11y.
- „Znam bibliotekę X" zamiast rozumowania o problemie (biblioteka to szczegół implementacji).
- Brak trade-offów — podawanie decyzji jak oczywistych pewników.
- Over-engineering ponad wymagania / brak priorytetyzacji (wszystko naraz, płytko).
- Monolog zamiast dialogu z rozmówcą.
- Pomijanie edge case'ów i stanów błędu.

## Pytania rekrutacyjne (przykładowe tematy)

1. **Zaprojektuj autocomplete/wyszukiwarkę.** — debounce, race, cache, a11y, stany (patrz wyżej).
2. **Zaprojektuj nieskończony feed.** — wirtualizacja, IO, cursor pagination, media, real-time.
3. **Zaprojektuj komponent tabeli z sortowaniem/filtrowaniem 100k wierszy.** — wirtualizacja, [worker](../05-dom-i-web-api/06-web-workers.md), stan w URL, wydajność.
4. **Zaprojektuj system powiadomień real-time.** — [SSE/WebSocket](../12-komunikacja-z-api/03-czas-rzeczywisty.md), reconnect, stan, badge, a11y.
5. **Zaprojektuj design system / bibliotekę komponentów.** — [architektura CSS/tokeny](../03-css/09-architektura-css.md), a11y, API komponentów, [dokumentacja](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md), wersjonowanie.

## Dalsza lektura

- [GreatFrontEnd — Front End System Design Playbook (RADIO)](https://www.greatfrontend.com/front-end-system-design-playbook)
- [Frontend at Scale — system design](https://frontendatscale.com/)
- „Frontend System Design" — materiały społeczności; ćwicz na głos z kimś
