# DevTools i debugowanie

> **Poziom:** 🟡 średni → 🔴 (profilowanie)
> **Wymagana wiedza:** [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md), [Zdarzenia](../05-dom-i-web-api/02-zdarzenia.md)

DevTools to najważniejsze narzędzie diagnostyczne frontend developera. Biegłość w nich odróżnia „dodam console.log i zgadnę" od „zdiagnozuję przyczynę w 5 minut". Ten plik to przegląd paneli pod kątem *co gdzie znajdziesz* — nie zastąpi godzin praktyki.

## Debugowanie zamiast `console.log`

**Breakpointy** biją logowanie: zatrzymują wykonanie i pozwalają zbadać **cały** stan.

- **Sources panel:** breakpoint klikiem na numerze linii; **conditional breakpoint** (tylko gdy warunek), **logpoint** (loguj bez modyfikacji kodu), breakpoint na zdarzeniu/zmianie DOM/`fetch`/wyjątku (**Pause on exceptions** — łapie błąd w momencie rzucenia, z pełnym [stackiem](../04-javascript/11-obsluga-bledow.md)).
- Podczas pauzy: **Scope** (zmienne lokalne/closure — zobacz [domknięcia](../04-javascript/03-scope-hoisting-closures.md) na żywo), **Call Stack**, **Watch**, step over/into/out, wykonywanie w konsoli w bieżącym kontekście.
- `debugger;` w kodzie = programowy breakpoint.
- Source maps ([build](./03-bundlery-i-proces-budowania.md)) → debugujesz oryginalny TS, nie zminifikowany bundle.

**Konsola** poza logowaniem: `console.table` (tablice obiektów), `.group`, `.time/.timeEnd`, `.trace`, `.assert`, `$0` (wybrany element), `$$('sel')` (=querySelectorAll), `copy(obj)`, monitorowanie zdarzeń.

## Elements — DOM, CSS, a11y

- Inspekcja i edycja [DOM](../05-dom-i-web-api/01-dom-manipulacja.md)/stylów na żywo; **Styles** pokazuje kaskadę, przekreślone (przegrane) deklaracje i **dlaczego** — bezcenne przy [specificity](../03-css/01-kaskada-specyficznosc.md).
- **Computed** — finalne wartości; **wymuszanie stanów** (`:hover`, `:focus`) do stylowania trudnych stanów.
- **Layout** — inspektor [flex/grid](../03-css/05-grid.md), overlay.
- **Accessibility** — [drzewo dostępności](../10-dostepnosc/02-aria-i-semantyka.md), computed name/role, kontrast w color pickerze ([a11y w praktyce](../10-dostepnosc/03-a11y-w-praktyce.md)).

## Network — cała komunikacja

- Lista requestów: status, typ, rozmiar, **czas** (waterfall z fazami: DNS/TCP/TLS/TTFB/download — [jak działa internet](../01-fundamenty-sieci/01-jak-dziala-internet.md)), **Priority**, initiator.
- **Throttling** — Slow 4G / offline: testuj realne warunki, nie biurowy światłowód ([wydajność](../09-wydajnosc/01-core-web-vitals.md)).
- Nagłówki żądań/odpowiedzi, payload, podgląd JSON — debug [API](../12-komunikacja-z-api/01-rest.md), [CORS](../11-bezpieczenstwo/02-mechanizmy-obronne.md), [cache](../09-wydajnosc/04-strategie-cachowania.md) (kolumna „from cache", nagłówki cache).
- „Disable cache" przy otwartych DevTools; kopiuj jako cURL/fetch.

## Performance — profilowanie runtime

Serce diagnozy [janku i INP](../09-wydajnosc/03-optymalizacja-dzialania.md):

- **Nagranie** pokazuje: flame chart **main threadu** (co zajmuje CPU), **Long Tasks** (czerwone rogi — blokada >50 ms), FPS, layout shifts, markery [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md).
- **CPU throttling** (4–6×) — symuluj słabszy sprzęt (rzeczywistość użytkownika ≠ Twój laptop).
- Znajdź drogie funkcje (bottom-up/call tree), wymuszone [reflow](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md) (fioletowe „Layout"/„Recalculate Style"), za częste rendery.
- Nowszy: panel **Performance** z nagraniami CWV z realnych interakcji; zakładka „Insights".

## Memory — wycieki

Diagnostyka [memory leaków](../04-javascript/12-zaawansowane.md):

- **Heap snapshot** — porównaj dwa (przed/po akcji): co przyrosło i nie zniknęło; szukaj **Detached** (oderwane węzły DOM trzymane w JS).
- **Allocation timeline** — kiedy i co alokuje.
- Objaw leaka: pamięć rośnie z użyciem i nie spada po „sprzątających" akcjach.

## Application i inne panele

- **Application:** [storage](../05-dom-i-web-api/04-przechowywanie-danych.md) (cookies z flagami, localStorage, IndexedDB, Cache), **[Service Workers](../08-architektura-aplikacji/02-spa-mpa-pwa.md)** (stan, update, unregister — ratunek przy „starej wersji"), manifest PWA.
- **Lighthouse** — audyt wydajności/a11y/SEO/PWA ([CWV](../09-wydajnosc/01-core-web-vitals.md)).
- **Rozszerzenia frameworków** (React/Vue DevTools) — drzewo komponentów, props/stan, **profiler renderów** ([reaktywność](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md)): co i czemu się renderuje.
- **Device toolbar** — emulacja urządzeń/DPR (pomocne, ale nie zastąpi realnego urządzenia — [responsywność](../03-css/06-responsywnosc.md)).

## Pułapki i częste błędy

- `console.log`-driven debugging tam, gdzie breakpoint pokazałby stan od razu.
- Profilowanie bez CPU/network throttlingu — „u mnie szybko".
- Testowanie tylko w Chrome — silniki różnią się ([przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)); sprawdź WebKit (Safari) i Firefox.
- Ignorowanie ostrzeżeń w konsoli (a11y, deprecation, CSP, React keys).
- Debugowanie zminifikowanego kodu bez source maps.
- Zapomniany [Service Worker](../08-architektura-aplikacji/02-spa-mpa-pwa.md) serwujący starą wersję — sprawdź Application → SW.

## Pytania rekrutacyjne

1. **Jak debugujesz bez `console.log`?** — breakpointy (conditional/logpoint/pause on exception), scope, call stack.
2. **Strona się zacina — których paneli użyjesz?** — Performance (long tasks, main thread) + throttling; ewentualnie profiler frameworka.
3. **Jak zdiagnozujesz memory leak?** — heap snapshots, detached nodes, wzrost pamięci.
4. **Jak sprawdzisz problem z requestem/CORS/cache?** — Network: nagłówki, status, waterfall, from-cache.
5. **Jak przetestujesz na wolnym łączu/słabym CPU?** — throttling sieci i CPU w DevTools.

## Dalsza lektura

- [Chrome DevTools — dokumentacja](https://developer.chrome.com/docs/devtools)
- [Chrome DevTools: Analyze runtime performance / Fix memory problems](https://developer.chrome.com/docs/devtools/performance)
- [Firefox DevTools](https://firefox-source-docs.mozilla.org/devtools-user/)
