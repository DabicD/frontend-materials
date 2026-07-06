# Optymalizacja ładowania

> **Poziom:** 🟡 średni → 🔴 zaawansowany
> **Wymagana wiedza:** [Core Web Vitals](./01-core-web-vitals.md), [Protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md), [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)

Ładowanie strony to optymalizacja **critical rendering path**: mniej bajtów, we właściwej kolejności, z wyprzedzeniem. Trzy dźwignie: (1) zmniejsz zasoby, (2) ustaw priorytety, (3) opóźnij niekrytyczne. Wszystko poniżej to warianty tych trzech ruchów.

## JavaScript — zwykle koszt nr 1

JS boli podwójnie: transfer **i** parsowanie/wykonanie na CPU (mobile!). Kolejność działań:

1. **Mniej kodu:** audyt bundle (`rollup-plugin-visualizer`/`webpack-bundle-analyzer` — co properly zajmuje miejsce), wymiana ciężkich zależności (moment→date-fns/Intl, lodash→metody natywne), [tree shaking](../14-narzedzia/03-bundlery-i-proces-budowania.md) (importy nazwane, uwaga na barrel files).
2. **Code splitting** — [dynamiczny import](../04-javascript/10-moduly.md): per trasa ([routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)), per ciężki komponent (edytor, wykresy, mapa), per interakcja (modal ładowany przy otwarciu).
3. **Defer wykonania:** `defer`/`type="module"` ([struktura dokumentu](../02-html/01-struktura-dokumentu.md)); third-parties ładuj po interakcji/idle (Partytown przenosi do workera).
4. **Mniej JS z natury:** [SSR/SSG/islands/Server Components](../08-architektura-aplikacji/01-wzorce-renderowania.md).

## CSS i fonty

- CSS jest render-blocking: trzymaj **krytyczny CSS mały** (inline krytycznej części przy dużych stronach), resztę ładuj później; usuwaj martwy CSS (coverage).
- **Fonty:** `font-display: swap` (tekst widoczny od razu) lub `optional`; `preload` dla kluczowego fontu; subsetting (tylko potrzebne znaki); formaty woff2; fallback z dopasowanymi metrykami (`size-adjust`) przeciw [CLS](./01-core-web-vitals.md). Fonty systemowe = zero kosztu.

## Obrazy i media

Największe bajty strony ([multimedia — dom tematu](../02-html/04-multimedia-i-grafika.md)): nowoczesne formaty (AVIF/WebP), `srcset/sizes`, wymiary przeciw CLS, `loading="lazy"` poza viewportem — i **nigdy** dla LCP. Dla hero: `fetchpriority="high"` (+ ewentualnie `<link rel="preload" as="image">`).

## Priorytety i resource hints

Przeglądarka sama nadaje priorytety (CSS/fonty wysoko, obrazy poza ekranem nisko) — Ty korygujesz:

```html
<link rel="preconnect" href="https://api.example.com" />       <!-- DNS+TCP+TLS z wyprzedzeniem -->
<link rel="dns-prefetch" href="https://cdn.example.com" />      <!-- sam DNS (tani fallback) -->
<link rel="preload" href="/fonts/main.woff2" as="font" crossorigin />  <!-- "będzie potrzebne NA PEWNO, teraz" -->
<link rel="prefetch" href="/next-page-chunk.js" />               <!-- "może za chwilę" — idle, niski priorytet -->
<link rel="modulepreload" href="/app.js" />                      <!-- moduły ESM z zależnościami -->
<img fetchpriority="high" … />  /  fetch(url, { priority: 'low' })
```

- **preload ≠ prefetch:** bieżąca nawigacja vs następna; preload nadużyty **kradnie pasmo** krytycznym zasobom.
- **Speculation Rules API** — prerender/prefetch całych podstron przy hover/viewport (następca `<link rel="prerender">`); MPA z tym potrafi nawigować „natychmiast".
- **bfcache** — back/forward z pamięci: nie blokuj go (`unload` listener, `Cache-Control: no-store` dla HTML) — darmowa „wydajność" najczęstszej nawigacji.

## Waterfall i łańcuchy zależności

Wydajność zabijają **sekwencyjne odkrycia**: HTML → CSS → font (odkryty w CSS) → render; albo JS → fetch → kolejny fetch. Zasady:

- Krytyczne zasoby **widoczne w HTML** (preload scanner — [jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)), nie odkrywane przez JS/CSS.
- Dane dla widoku startuj **równolegle z kodem** (loader/preload danych), nie po hydracji ([wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)).
- Analizuj w DevTools Network: waterfall, kolumna Priority, initiator ([DevTools](../14-narzedzia/05-devtools.md)).

## Kompresja i transfer

- **Brotli** (statyki: poziom 11 w buildzie) / gzip — tekstowe zasoby; weryfikuj `Content-Encoding`.
- Minifikacja JS/CSS/HTML ([proces budowania](../14-narzedzia/03-bundlery-i-proces-budowania.md)).
- [HTTP/2+](../01-fundamenty-sieci/02-protokol-http.md) — multipleksacja zamiast konkatenacji wszystkiego; [CDN](../01-fundamenty-sieci/04-domeny-hosting-cdn.md) i [cache](./04-strategie-cachowania.md) — osobne pliki.

## Pułapki i częste błędy

- Lazy loading obrazu LCP (klasyk nr 1).
- `preload` dziesiątek zasobów — priorytetowy korek; preload to skalpel.
- Splitting na 200 mikro-chunków — kaskady requestów; balans (grupowanie tras).
- Trzecie strony (analityka, czaty, tagi) nieaudytowane — potrafią ważyć więcej niż aplikacja; ładuj leniwie, mierz, usuwaj.
- Optymalizacja bundle'a przy 4 MB obrazka hero — zaczynaj od największych bajtów (Network → sort by size).
- Brak `dimensions`/placeholderów → CLS mimo szybkiego ładowania.

## Pytania rekrutacyjne

1. **Strona ładuje się wolno — plan działania?** — zmierz (field→lab), waterfall, największe bajty/łańcuchy, potem dźwignie wg kosztu.
2. **preload vs prefetch vs preconnect?** — teraz-na-pewno vs później-może vs samo połączenie.
3. **Strategie code splittingu?** — trasa/komponent/interakcja; jak unikać kaskad.
4. **Jak zoptymalizujesz webfonty?** — woff2, subset, font-display, preload, fallback metrics.
5. **Co to bfcache i co go blokuje?** — natychmiastowy back/forward; unload, no-store.

## Dalsza lektura

- [web.dev: Fast load times / Learn Performance](https://web.dev/learn/performance)
- [MDN: Resource hints (preload/prefetch/preconnect)](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/rel/preload)
- [Chrome: Speculation Rules](https://developer.chrome.com/docs/web-platform/prerender-pages)
