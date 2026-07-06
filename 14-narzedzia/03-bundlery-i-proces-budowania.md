# Bundlery i proces budowania

> **Poziom:** 🟡 średni → 🔴 (optymalizacja)
> **Wymagana wiedza:** [Moduły](../04-javascript/10-moduly.md), [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)

Nowoczesny frontend nie wysyła do przeglądarki tego, co piszesz — kod przechodzi przez **build**: transpilację, bundling, minifikację, optymalizacje. Zrozumienie tego procesu tłumaczy, skąd biorą się chunki, source mapy i czemu „u mnie działa, na produkcji nie".

## Po co w ogóle bundler

Piszesz kod w setkach [modułów](../04-javascript/10-moduly.md), z TS/JSX, importujesz CSS i obrazy, używasz składni, której stare przeglądarki nie znają. Bundler:

1. **Buduje graf zależności** od punktów wejścia (kto co importuje).
2. **Transpiluje** — TS/JSX/nowy JS → JS zrozumiały dla targetów ([browserslist](../03-css/10-preprocesory-i-narzedzia.md)).
3. **Bundluje** — łączy moduły w pliki (chunki), rozwiązuje ścieżki, obsługuje nie-JS (CSS, obrazy, fonty).
4. **Optymalizuje** — tree shaking, minifikacja, code splitting, hashe w nazwach ([cache busting](../09-wydajnosc/04-strategie-cachowania.md)).

## Krajobraz narzędzi

- **Vite** — de facto standard nowych projektów. Dev: natywny ESM (przeglądarka ładuje moduły, transformacja on-demand — **błyskawiczny start** niezależny od wielkości projektu) + esbuild. Produkcja: bundle przez Rollup. Prosty config, HMR out of the box.
- **webpack** — dojrzały, wszechstronny, ogromny ekosystem loaderów/pluginów; wolniejszy, config cięższy; wciąż w wielu (starszych) projektach i przy [Module Federation](../08-architektura-aplikacji/03-microfrontends-i-monorepo.md).
- **esbuild** (Go) / **SWC** (Rust) / **Rolldown** (Rust, przyszły silnik Vite) — ultraszybkie transpilery/bundlery, napędzają inne narzędzia. **Turbopack** — następca webpacka w Next.
- **Rollup** — świetny do **bibliotek** (czysty output, tree shaking).

Trend: silniki przepisywane na Go/Rust (rzędy wielkości szybciej niż stary JS-owy tooling).

## Kluczowe pojęcia

**Transpilacja** — zamiana składni na starszą/zgodną. **Babel** (klasyk, wtyczkowy) lub szybsze SWC/esbuild. Uwaga: transpilacja składni ≠ **polyfille** (nowe API jak `Promise`, `fetch` wymagają runtime'owego uzupełnienia — core-js; dobierane wg [browserslist](../03-css/10-preprocesory-i-narzedzia.md)). Nadmiar transpilacji/polyfilli = zbędna waga dla nowoczesnych przeglądarek.

**Tree shaking** — usuwanie martwego (nieimportowanego) kodu; działa dzięki **statyczności [ESM](../04-javascript/10-moduly.md)**. Wrogowie: side effects (pole `"sideEffects"` w package.json), CommonJS, [barrel files](../04-javascript/10-moduly.md), importy całych bibliotek zamiast wybiórczych.

**Code splitting** — dzielenie bundle'a na chunki ładowane na żądanie ([dynamiczny import](../04-javascript/10-moduly.md)): per [trasa](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md), per komponent, vendor chunk. Fundament [optymalizacji ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md).

**Minifikacja** — usuwanie białych znaków, skracanie nazw, dead code elimination (esbuild/terser/SWC). **Kompresja** (Brotli/gzip) to osobny krok na serwerze ([optymalizacja](../09-wydajnosc/02-optymalizacja-ladowania.md)).

**Content hashing** — `app.3f2a1b.js`; zmiana treści = nowa nazwa → agresywny cache statyk + poprawna inwalidacja ([cache](../09-wydajnosc/04-strategie-cachowania.md)).

## HMR i dev server

**HMR (Hot Module Replacement)** — podmiana zmienionego modułu **bez przeładowania strony i utraty stanu** aplikacji. Klucz do szybkiej pętli developerskiej; Vite robi to natywnie i granularnie dzięki ESM.

Dev server dokłada: proxy do API (omija [CORS](../11-bezpieczenstwo/02-mechanizmy-obronne.md) lokalnie), zmienne środowiskowe, obsługę `.env` ([hosting](../16-devops-dla-frontendu/02-hosting-i-deployment.md)).

## Source maps

Mapują zminifikowany/transpilowany kod z powrotem na oryginalne źródła — debugowanie i czytelne [stack trace'y](../04-javascript/11-obsluga-bledow.md) w produkcji ([monitoring](../16-devops-dla-frontendu/03-monitoring-i-obserwabilnosc.md)). W produkcji zwykle generowane, ale **nieudostępniane publicznie** (upload do narzędzia błędów) — inaczej ujawniasz kod źródłowy.

## Analiza i budżety

- **Bundle analyzer** (rollup-plugin-visualizer / webpack-bundle-analyzer) — mapa, **co** zajmuje miejsce; pierwszy krok każdej optymalizacji wagi (często zaskakująca duża zależność).
- **Budżety** wielkości w [CI](../16-devops-dla-frontendu/01-ci-cd.md) — blokuj PR-y pęczniejące ponad limit.
- Świadomość, że **TS tylko transpiluje** (nie sprawdza typów w buildzie) — walidacja to osobny `tsc --noEmit` ([TS w praktyce](../06-typescript/05-typescript-w-praktyce.md)).

## Pułapki i częste błędy

- Import całej biblioteki (`import _ from 'lodash'`) zamiast wybiórczo/natywnie — psuje tree shaking, puchnie bundle.
- Barrel files / side effects blokujące tree shaking.
- Publiczne source mapy w produkcji — wyciek kodu.
- Przesadna transpilacja (target ES5 w 2026) + zbędne polyfille — waga i wolniejszy kod na nowoczesnych przeglądarkach.
- Zmienne środowiskowe: wszystko w bundlu jest **jawne** — żadnych sekretów w `.env` frontendu ([bezpieczeństwo](../11-bezpieczenstwo/01-ataki-na-frontend.md)); tylko prefiksowane (`VITE_`) trafiają do klienta.
- Optymalizacja bez `analyzer` — zgadywanie.

## Pytania rekrutacyjne

1. **Co robi bundler i po co?** — graf, transpilacja, bundling, optymalizacje; problemy, które rozwiązuje.
2. **Czemu Vite ma szybszy dev niż webpack?** — natywny ESM on-demand + esbuild vs bundlowanie całości.
3. **Jak działa tree shaking i co go psuje?** — statyczny ESM; side effects, CJS, barrel, importy całości.
4. **Transpilacja vs polyfill?** — składnia vs runtime API; rola browserslist.
5. **Do czego source maps i czy publikować je w produkcji?** — debug/stack trace; nie publicznie (upload do error trackera).
6. **Jak zabierasz się za redukcję wagi bundle'a?** — analyzer → największe pozycje → splitting/wymiana zależności/lazy; budżety w CI.

## Dalsza lektura

- [Vite — Why Vite / Features](https://vite.dev/guide/why.html)
- [web.dev: Reduce JavaScript payloads with code splitting](https://web.dev/articles/reduce-javascript-payloads-with-code-splitting)
- [The Deep Dive into Modern Bundlers (różne, np. esbuild docs)](https://esbuild.github.io/)
