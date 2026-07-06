# 🎯 Skarbnica wiedzy Frontend Developera

Kompletna, uporządkowana baza wiedzy pokrywająca frontend **od absolutnych podstaw po tematy senior-level** — pomyślana jako narzędzie do nadrobienia braków i przygotowania do rekrutacji na wymarzoną pracę.

Materiał jest **framework-agnostic**: skupia się na wiedzy ogólnej i koncepcyjnej (JavaScript, przeglądarka, sieć, architektura, wzorce), która przenosi się między React, Vue, Angular czy Svelte i starzeje wolniej niż konkretne biblioteki.

## Jak korzystać z tej skarbnicy

- **Struktura:** 17 tematycznych folderów, w każdym pliki numerowane wg kolejności nauki. Zaczynaj od niższych numerów.
- **Zero duplikacji:** każdy koncept ma **jedno miejsce** wyjaśnienia (plik-dom); inne pliki **linkują** do niego zamiast powtarzać. Podążaj za linkami `[tekst](ścieżka)`, by zgłębić prerekwizyty.
- **Format każdego pliku:** poziom trudności → wymagana wiedza → wyjaśnienia z przykładami kodu → **Pułapki i częste błędy** → **Pytania rekrutacyjne** (z odpowiedziami) → **Dalsza lektura** (MDN, web.dev, specyfikacje).
- **Legenda poziomów:** 🟢 podstawowy · 🟡 średni · 🔴 zaawansowany/senior.
- **Przygotowanie do rozmów:** [17-rekrutacja](#17--rekrutacja) — a zwłaszcza [bank pytań teoretycznych](./17-rekrutacja/02-pytania-teoretyczne.md), który indeksuje pytania z całej skarbnicy.

## Sugerowana ścieżka nauki

1. **Fundamenty platformy** → 01 (sieć) → 02 (HTML) → 03 (CSS)
2. **Serce zawodu** → 04 (JavaScript — największa i najważniejsza sekcja) → 05 (DOM i Web API)
3. **Standard zawodowy** → 06 (TypeScript)
4. **Aplikacje** → 07 (frameworki koncepcyjnie) → 08 (architektura)
5. **Jakość produktu** → 09 (wydajność) → 10 (dostępność) → 11 (bezpieczeństwo)
6. **Dane i testy** → 12 (komunikacja z API) → 13 (testowanie)
7. **Warsztat i proces** → 14 (narzędzia) → 15 (inżynieria) → 16 (DevOps)
8. **Rekrutacja** → 17 (proces, pytania, zadania, system design) — wracaj tu przez cały czas nauki

> Jeśli gonią Cię rozmowy: przejrzyj [bank pytań teoretycznych](./17-rekrutacja/02-pytania-teoretyczne.md) „na głos", oznacz luki i doczytaj wskazane pliki-domy. Priorytet dla senior: 04, 06, 07, 08, 09, 11.

---

## Mapa treści

### 01 · Fundamenty sieci
Jak działa warstwa, na której stoi cały frontend.
- [Jak działa internet](./01-fundamenty-sieci/01-jak-dziala-internet.md) — DNS, TCP/UDP, TLS, „co się dzieje po wpisaniu URL"
- [Protokół HTTP](./01-fundamenty-sieci/02-protokol-http.md) — metody, statusy, nagłówki, wersje, **HTTP caching**
- [Jak działa przeglądarka](./01-fundamenty-sieci/03-jak-dziala-przegladarka.md) — critical rendering path, reflow/repaint
- [Domeny, hosting i CDN](./01-fundamenty-sieci/04-domeny-hosting-cdn.md) — URL, origin, rekordy DNS, edge

### 02 · HTML
- [Struktura dokumentu](./02-html/01-struktura-dokumentu.md) — doctype, head, async/defer
- [Semantyka](./02-html/02-semantyka.md) — znaczniki, landmarki, `<a>` vs `<button>`
- [Formularze](./02-html/03-formularze.md) — walidacja natywna, autocomplete, UX
- [Multimedia i grafika](./02-html/04-multimedia-i-grafika.md) — responsywne obrazy, SVG vs Canvas
- [SEO i metadane](./02-html/05-seo-i-metadane.md) — Open Graph, JSON-LD, JS a indeksacja

### 03 · CSS
- [Kaskada, specificity, dziedziczenie](./03-css/01-kaskada-specyficznosc.md)
- [Box model i jednostki](./03-css/02-box-model-i-jednostki.md)
- [Układ dokumentu: display, position, z-index](./03-css/03-uklad-dokumentu.md)
- [Flexbox](./03-css/04-flexbox.md) · [Grid](./03-css/05-grid.md)
- [Responsywność](./03-css/06-responsywnosc.md) — media/container queries, clamp
- [Animacje i transformacje](./03-css/07-animacje-i-transformacje.md)
- [Nowoczesny CSS](./03-css/08-nowoczesny-css.md) — custom properties, `:has()`, `<dialog>`
- [Architektura CSS](./03-css/09-architektura-css.md) — BEM, Modules, Tailwind
- [Preprocesory i narzędzia](./03-css/10-preprocesory-i-narzedzia.md) — Sass, PostCSS

### 04 · JavaScript
Rdzeń zawodu — największa sekcja.
- [Typy i zmienne](./04-javascript/01-typy-i-zmienne.md) · [Funkcje](./04-javascript/02-funkcje.md)
- [Scope, hoisting, **closures**](./04-javascript/03-scope-hoisting-closures.md) · [`this` i kontekst](./04-javascript/04-this-i-kontekst.md)
- [Obiekty i prototypy](./04-javascript/05-obiekty-i-prototypy.md) · [Kolekcje i iteracja](./04-javascript/06-kolekcje-i-iteracja.md)
- [Składnia ES6+](./04-javascript/07-skladnia-es6-plus.md) · [**Asynchroniczność** (Promises)](./04-javascript/08-asynchronicznosc.md)
- [**Event loop**](./04-javascript/09-event-loop.md) · [Moduły (ESM/CJS)](./04-javascript/10-moduly.md)
- [Obsługa błędów](./04-javascript/11-obsluga-bledow.md) · [Zaawansowane (Proxy, GC, pamięć)](./04-javascript/12-zaawansowane.md)
- [Wzorce projektowe w JS](./04-javascript/13-wzorce-projektowe-js.md)

### 05 · DOM i Web API
- [DOM — manipulacja](./05-dom-i-web-api/01-dom-manipulacja.md) · [Zdarzenia](./05-dom-i-web-api/02-zdarzenia.md)
- [Fetch — HTTP w praktyce](./05-dom-i-web-api/03-fetch-i-http-w-praktyce.md) · [Przechowywanie danych](./05-dom-i-web-api/04-przechowywanie-danych.md)
- [Obserwery (Intersection/Resize/Mutation)](./05-dom-i-web-api/05-observery.md) · [Web Workers](./05-dom-i-web-api/06-web-workers.md)
- [Przydatne API przeglądarki](./05-dom-i-web-api/07-przydatne-api-przegladarki.md)

### 06 · TypeScript
- [Podstawy typowania](./06-typescript/01-podstawy-typowania.md) · [Interfejsy, unie, narrowing](./06-typescript/02-interfejsy-i-typy.md)
- [Funkcje i generyki](./06-typescript/03-funkcje-i-generyki.md) · [Typy zaawansowane](./06-typescript/04-typy-zaawansowane.md)
- [TypeScript w praktyce](./06-typescript/05-typescript-w-praktyce.md) — tsconfig, walidacja, migracja

### 07 · Frameworki koncepcyjnie
Koncepty wspólne dla wszystkich frameworków.
- [Po co frameworki](./07-frameworki-koncepcyjnie/01-po-co-frameworki.md) · [Model komponentowy](./07-frameworki-koncepcyjnie/02-model-komponentowy.md)
- [Modele reaktywności (VDOM, signals)](./07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md) · [Zarządzanie stanem](./07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)
- [Routing po stronie klienta](./07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md) · [Przegląd frameworków](./07-frameworki-koncepcyjnie/06-przeglad-frameworkow.md)

### 08 · Architektura aplikacji
- [Wzorce renderowania (CSR/SSR/SSG/ISR)](./08-architektura-aplikacji/01-wzorce-renderowania.md)
- [SPA, MPA, PWA + Service Worker](./08-architektura-aplikacji/02-spa-mpa-pwa.md)
- [Microfrontends i monorepo](./08-architektura-aplikacji/03-microfrontends-i-monorepo.md)
- [Struktura projektu i granice modułów](./08-architektura-aplikacji/04-struktura-projektu.md)

### 09 · Wydajność
- [Core Web Vitals i mierzenie](./09-wydajnosc/01-core-web-vitals.md)
- [Optymalizacja ładowania](./09-wydajnosc/02-optymalizacja-ladowania.md)
- [Optymalizacja działania (**debounce/throttle**, wirtualizacja)](./09-wydajnosc/03-optymalizacja-dzialania.md)
- [Strategie cache'owania](./09-wydajnosc/04-strategie-cachowania.md)

### 10 · Dostępność
- [Podstawy a11y (WCAG, EAA)](./10-dostepnosc/01-podstawy-a11y.md)
- [ARIA i drzewo dostępności](./10-dostepnosc/02-aria-i-semantyka.md)
- [Dostępność w praktyce (focus, kontrast, testy)](./10-dostepnosc/03-a11y-w-praktyce.md)

### 11 · Bezpieczeństwo
- [Ataki na frontend (XSS, CSRF, supply chain)](./11-bezpieczenstwo/01-ataki-na-frontend.md)
- [Mechanizmy obronne (SOP, CORS, CSP)](./11-bezpieczenstwo/02-mechanizmy-obronne.md)
- [Uwierzytelnianie i autoryzacja](./11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)

### 12 · Komunikacja z API
- [REST](./12-komunikacja-z-api/01-rest.md) · [GraphQL](./12-komunikacja-z-api/02-graphql.md)
- [Czas rzeczywisty (WebSocket, SSE)](./12-komunikacja-z-api/03-czas-rzeczywisty.md)
- [Wzorce pracy z danymi (cache, race conditions)](./12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)

### 13 · Testowanie
- [Strategia testowania (piramida vs trofeum)](./13-testowanie/01-strategia-testowania.md)
- [Testy jednostkowe](./13-testowanie/02-testy-jednostkowe.md)
- [Testy komponentów i integracyjne](./13-testowanie/03-testy-komponentow-i-integracyjne.md)
- [Testy E2E i specjalistyczne](./13-testowanie/04-testy-e2e-i-specjalistyczne.md)

### 14 · Narzędzia
- [Git](./14-narzedzia/01-git.md) · [Pakiety i zależności](./14-narzedzia/02-pakiety-i-zaleznosci.md)
- [Bundlery i proces budowania](./14-narzedzia/03-bundlery-i-proces-budowania.md) · [Jakość kodu (ESLint/Prettier)](./14-narzedzia/04-jakosc-kodu.md)
- [DevTools i debugowanie](./14-narzedzia/05-devtools.md)

### 15 · Inżynieria oprogramowania
- [Czysty kod i zasady projektowe (SOLID)](./15-inzynieria-oprogramowania/01-czysty-kod.md)
- [Code review i współpraca](./15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md)
- [Praca z legacy i refaktoryzacja](./15-inzynieria-oprogramowania/03-praca-z-legacy.md)

### 16 · DevOps dla frontendu
- [CI/CD](./16-devops-dla-frontendu/01-ci-cd.md)
- [Hosting i deployment](./16-devops-dla-frontendu/02-hosting-i-deployment.md)
- [Monitoring i obserwowalność](./16-devops-dla-frontendu/03-monitoring-i-obserwabilnosc.md)

### 17 · Rekrutacja
- [Proces rekrutacyjny (CV, STAR, negocjacje)](./17-rekrutacja/01-proces-rekrutacyjny.md)
- [**Bank pytań teoretycznych**](./17-rekrutacja/02-pytania-teoretyczne.md) — indeks pytań z całej skarbnicy
- [Zadania praktyczne i live coding](./17-rekrutacja/03-zadania-praktyczne.md)
- [Frontend system design](./17-rekrutacja/04-frontend-system-design.md)

---

## Checklista postępów

Zaznaczaj ukończone sekcje (`[ ]` → `[x]`), by śledzić progres.

- [ ] 01 · Fundamenty sieci
- [ ] 02 · HTML
- [ ] 03 · CSS
- [ ] 04 · JavaScript ⭐ (rdzeń — poświęć najwięcej czasu)
- [ ] 05 · DOM i Web API
- [ ] 06 · TypeScript
- [ ] 07 · Frameworki koncepcyjnie
- [ ] 08 · Architektura aplikacji
- [ ] 09 · Wydajność
- [ ] 10 · Dostępność
- [ ] 11 · Bezpieczeństwo
- [ ] 12 · Komunikacja z API
- [ ] 13 · Testowanie
- [ ] 14 · Narzędzia
- [ ] 15 · Inżynieria oprogramowania
- [ ] 16 · DevOps dla frontendu
- [ ] 17 · Rekrutacja (wracaj przez cały czas)

---

## Filozofia nauki (przeczytaj raz)

1. **Fundamenty ponad frameworkiem.** Rozmowy i realna praca weryfikują JavaScript, przeglądarkę i sieć — nie znajomość API biblioteki. Braki w [closures](./04-javascript/03-scope-hoisting-closures.md), [event loopie](./04-javascript/09-event-loop.md) czy [`this`](./04-javascript/04-this-i-kontekst.md) wracają jako „magiczne" bugi.
2. **Ucz się jednego frameworka głęboko, resztę koncepcyjnie.** Koncepty z sekcji 07 przenoszą się wszędzie.
3. **Zrozumienie > zapamiętanie.** Sekcje „Pytania rekrutacyjne" sprawdzaj *odpowiadając na głos* — rozmówca drąży „dlaczego?", a wykuta formułka się nie obroni.
4. **Ucz się przez robienie.** Do każdego tematu zbuduj coś małego; „Dalsza lektura" prowadzi do źródeł (MDN, web.dev, specyfikacje).
5. **Pułapki to złoto.** Sekcje „Pułapki i częste błędy" to skondensowane doświadczenie — częściej odróżniają seniora niż znajomość składni.

Powodzenia. 💪
