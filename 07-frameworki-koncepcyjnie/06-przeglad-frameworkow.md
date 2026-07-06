# Przegląd frameworków — filozofie i różnice

> **Poziom:** 🟡 średni (przegląd rynku)
> **Wymagana wiedza:** [Modele reaktywności](./03-modele-reaktywnosci.md), [Model komponentowy](./02-model-komponentowy.md)

Wszystkie omawiane frameworki realizują te same koncepty (komponenty, reaktywność, routing, SSR) — różnią się **filozofią i kompromisami**. Ten plik daje mapę: co czym jest, czym się wyróżnia i jak czytać oferty pracy. Świadomie bez tutoriali składni — od tego są dokumentacje.

## Szybka mapa

| | Model reaktywności | Szablony | Charakter |
|---|---|---|---|
| **React** | VDOM, re-render + memo (+ React Compiler) | JSX (JS jako szablon) | biblioteka widoku + ogromny ekosystem |
| **Vue** | Proxy-reactivity + VDOM (kompilowane optymalizacje) | SFC: HTML-owe szablony (lub JSX) | framework progresywny, „środek drogi" |
| **Angular** | signals (nowe) / zone.js (legacy) | szablony HTML + dyrektywy | pełna platforma, batteries-included |
| **Svelte** | kompilator + runes (signals) | SFC, najmniej boilerplate'u | „znikający framework" |
| **Solid** | fine-grained signals, bez VDOM | JSX (wykonywany raz) | maksimum wydajności, mały rynek |

## React

- **Filozofia:** UI = funkcja stanu; wszystko jest JavaScriptem (JSX, logika, warunki — zwykły JS). Minimalny rdzeń, decyzje (router, stan, formularze) oddane ekosystemowi.
- **Konsekwencje:** największy rynek pracy i ekosystem; ale „React" w CV znaczy też: wybory architektoniczne robisz sam, a model re-renderów trzeba **rozumieć** (zasady hooków, zależności efektów, memoizacja — [reaktywność](./03-modele-reaktywnosci.md)).
- **Kierunek:** Server Components i frameworki fullstackowe (Next.js) — przesuwanie pracy na serwer ([wzorce renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md)); React Compiler automatyzuje optymalizacje.
- Meta-framework: **Next.js** (dominujący), Remix/React Router.

## Vue

- **Filozofia:** progresywność — od `<script>` na jednej stronie po pełne SPA; Single File Components (szablon+skrypt+style w jednym pliku); reaktywność Proxy „po prostu działa" (mutujesz stan — widok się aktualizuje).
- **Composition API** (dzisiejszy standard) — composables jak hooki, ale bez reguł kolejności; Options API w starszych kodach.
- **Konsekwencje:** łagodna krzywa wejścia, spójny oficjalny ekosystem (router, Pinia); rynek pracy mniejszy niż React, mocny w Azji/Europie.
- Meta-framework: **Nuxt**.

## Angular

- **Filozofia:** kompletna platforma z opiniami: DI, routing, formularze (template-driven i reactive), HTTP, testy — wszystko wbudowane i spójne; TypeScript od zawsze; struktura narzucona (dziś standalone components zamiast NgModules).
- **Konsekwencje:** świetny w dużych organizacjach (spójność między zespołami, długowieczność); cięższy start, mniej „modny". Nowoczesny Angular (signals, control flow `@if/@for`, zoneless) jest znacznie lżejszy koncepcyjnie niż jego reputacja.
- Rynek: enterprise, banki, korporacje.

## Svelte i Solid (świadomość trendów)

- **Svelte:** kompilator — komponenty kompilują się do imperatywnego JS bez runtime'owego diffowania; runes (Svelte 5) = signals z jawną składnią. Minimalny kod, świetny DX; meta-framework **SvelteKit**. Rynek rosnący, ale mały.
- **Solid:** JSX jak React, ale wykonywany raz + fine-grained signals — czołówka benchmarków. Znaczenie głównie koncepcyjne (kierunek, w którym poszły inne).
- Nisze warte kojarzenia: **Qwik** (resumability — zero hydracji), **Astro** (islands, strony treściowe — [wzorce renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md)), **htmx** (hipermedia zamiast SPA), **Web Components/Lit** (standard platformy, design systemy — [architektura CSS](../03-css/09-architektura-css.md): Shadow DOM).

## Jak wybierać (i jak o tym mówić na rozmowie)

Kryteria seniora — w tej kolejności:

1. **Zespół i rynek pracy** — kompetencje ludzi i dostępność kandydatów biją benchmarki.
2. **Charakter produktu** — treściowy (Astro/MPA/SSG) vs aplikacyjny (SPA/SSR); wymogi [SEO](../02-html/05-seo-i-metadane.md) i [wydajności](../09-wydajnosc/01-core-web-vitals.md).
3. **Ekosystem potrzebnych rozwiązań** — design system, biblioteki domenowe, wsparcie długoterminowe.
4. **Spójność organizacji** — jeden framework w firmie > „najlepszy" per projekt.

Benchmarki wydajności różnicują się na ekstremach; dla 90% aplikacji każda z opcji jest „wystarczająco szybka" — o wydajności decyduje architektura ładowania ([optymalizacja](../09-wydajnosc/02-optymalizacja-ladowania.md)), nie wybór frameworka.

**Umiejętność przenośna:** koncepty z tej sekcji + [JS](../04-javascript/01-typy-i-zmienne.md)/[TS](../06-typescript/01-podstawy-typowania.md)/[DOM](../05-dom-i-web-api/01-dom-manipulacja.md). Framework to składnia na wierzchu — ucz się jednego głęboko (rynek: najczęściej React), czytaj pozostałe koncepcyjnie.

## Pułapki i częste błędy

- Wojny religijne zamiast analizy kompromisów — czerwona flaga na rozmowach.
- „Znam 5 frameworków" powierzchownie zamiast 1 głęboko + koncepty.
- Wybór pod CV-driven development w projekcie komercyjnym.
- Ocena frameworka po benchmarku „todo list" zamiast po kosztach utrzymania.

## Pytania rekrutacyjne

1. **Porównaj React i Vue/Angular — filozofie, nie składnię.** — biblioteka+ekosystem vs progresywny vs platforma; modele reaktywności.
2. **Jak wybrałbyś framework dla nowego projektu?** — kryteria wyżej; pokaż proces, nie fanatyzm.
3. **Co to jest meta-framework i po co?** — SSR/SSG/routing/splitting na frameworku widoku (Next/Nuxt/SvelteKit).
4. **Czym różni się podejście Svelte od Reacta?** — kompilacja vs runtime; konsekwencje dla bundle i modelu pracy.
5. **Co z Twojej wiedzy przenosi się między frameworkami?** — koncepty tej sekcji; platforma webowa.

## Dalsza lektura

- [State of JS](https://stateofjs.com/) — coroczny puls ekosystemu
- Oficjalne tutoriale: [react.dev](https://react.dev/learn), [vuejs.org](https://vuejs.org/tutorial/), [angular.dev](https://angular.dev/tutorials), [svelte.dev](https://svelte.dev/tutorial)
- [Component party](https://component-party.dev/) — te same wzorce we wszystkich frameworkach obok siebie
