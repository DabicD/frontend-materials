# Responsywność (RWD)

> **Poziom:** 🟢 podstawowy → 🟡 średni (container queries)
> **Wymagana wiedza:** [Box model i jednostki](./02-box-model-i-jednostki.md), [Flexbox](./04-flexbox.md), [Grid](./05-grid.md)

Responsive Web Design = jedna strona działająca na każdym rozmiarze ekranu. Nowoczesne podejście to nie „trzy wersje strony pod trzy breakpointy", tylko **layouty z natury płynne** (flex/grid/clamp), a media queries jako korekta tam, gdzie płynność nie wystarcza.

## Fundament: viewport meta

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

Bez tego mobile udaje ekran ~980px ([struktura dokumentu](../02-html/01-struktura-dokumentu.md)). Nie dodawaj `user-scalable=no`/`maximum-scale=1` — blokowanie zoomu to bariera [dostępności](../10-dostepnosc/03-a11y-w-praktyce.md).

## Media queries

```css
/* Mobile-first: bazowe style = mobile, media queries dokładają dla większych */
.layout { display: block; }

@media (min-width: 48rem) {          /* zamiast px — rem respektuje zoom czcionki */
  .layout { display: grid; grid-template-columns: 280px 1fr; }
}

/* Nowa składnia zakresów */
@media (48rem <= width < 80rem) { … }
```

- **Mobile-first** (`min-width`) to standard: prostsza baza, style „dokładane" a nie „odkręcane".
- **Breakpointy z treści, nie z urządzeń** — dodaj breakpoint tam, gdzie layout się psuje, nie „bo iPhone ma 390px".
- Media queries to nie tylko szerokość:

```css
@media (orientation: landscape) { … }
@media (hover: hover) and (pointer: fine) { … }  /* prawdziwy hover — nie dotyk */
@media (prefers-color-scheme: dark) { … }         /* dark mode */
@media (prefers-reduced-motion: reduce) { … }     /* ograniczenie animacji */
```

`prefers-reduced-motion` i `prefers-color-scheme` to dziś obowiązkowy standard, nie bajer.

## Container queries — responsywność komponentu

Media queries pytają o **viewport**; komponent w sidebarze i w treści ma jednak różną szerokość przy tym samym viewporcie. **Container queries** pytają o rozmiar **kontenera**:

```css
.sidebar, .main { container-type: inline-size; }   /* rejestracja kontenera */

.card { display: flex; flex-direction: column; }
@container (min-width: 400px) {
  .card { flex-direction: row; }                    /* karta pozioma, GDY MA MIEJSCE */
}
```

- Jednostki kontenera: `cqw`, `cqi` (1% szerokości kontenera) itd.
- Kontenery można nazywać: `container: card / inline-size`, potem `@container card (…)`.
- To zmiana paradygmatu dla design systemów: komponent sam wie, jak się ułożyć — niezależnie od strony, na której żyje.
- Są też **style queries** (`@container style(--variant: compact)`) — reagowanie na wartość custom property.

## Płynne wartości: `clamp()` i fluid typography

Zamiast skokowych zmian w breakpointach — wartości płynące z rozmiarem:

```css
h1 { font-size: clamp(1.75rem, 1.2rem + 2.5vw, 3rem); }
.section { padding-block: clamp(2rem, 5vw, 6rem); }
```

`clamp(min, preferowana, max)` — preferowana zwykle miksem `rem + vw` (czysty `vw` nie reaguje na zoom tekstu — problem a11y). Kalkulatory: „fluid type scale".

## Strategie layoutu bez media queries

Duża część responsywności wychodzi „za darmo" z layoutu:

- `repeat(auto-fill, minmax(250px, 1fr))` — [grid](./05-grid.md): kolumny same się dokładają.
- `flex-wrap: wrap` + `flex: 1 1 250px` — [flexbox](./04-flexbox.md): zawijanie z minimalną szerokością.
- `max-width: 65ch` dla tekstu; `width: min(90%, 1200px)` dla kontenera strony.
- Obrazy: `max-width: 100%; height: auto` + `srcset` ([multimedia](../02-html/04-multimedia-i-grafika.md)).

## Responsywność to nie tylko szerokość

- **Dotyk:** cele dotykowe min. ~44×44px; brak stanów `:hover` jako jedynego nośnika informacji (media query `hover: none`).
- **Wydajność:** mobile = słabszy CPU i sieć — [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md) mierz na mobile.
- **Wysokość:** klawiatura ekranowa, dynamiczny pasek adresu (`dvh` — [jednostki](./02-box-model-i-jednostki.md)), notche (`env(safe-area-inset-*)`).

## Pułapki i częste błędy

- Desktop-first z lawiną `max-width` odkręcających style.
- Breakpointy w px pod konkretne telefony (`@media (max-width: 390px)` „bo iPhone") — projektuj pod treść.
- Ukrywanie treści na mobile (`display: none`) zamiast przeprojektowania — użytkownik mobile nie jest gorszy; a ukryty DOM i tak się ładuje.
- Hover jako jedyny sposób dotarcia do akcji (menu, tooltips) — na dotyku nieosiągalne.
- Testowanie tylko przez zwężanie okna desktopowej przeglądarki — emulacja ≠ prawdziwy dotyk, DPR i klawiatura ekranowa.
- `100vh` na mobile (pasek adresu) — używaj `dvh`/`svh`.

## Pytania rekrutacyjne

1. **Mobile-first vs desktop-first — co i dlaczego?** — baza dla najprostszego widoku, `min-width` dokłada złożoność; mniej odkręcania.
2. **Media queries vs container queries?** — viewport vs rozmiar kontenera; komponenty wielokrotnego użytku potrzebują CQ.
3. **Jak zrobić płynną typografię?** — `clamp()` z miksem rem+vw; czemu nie czysty vw (zoom).
4. **Jak obsłużyć dark mode i reduced motion?** — `prefers-color-scheme`, `prefers-reduced-motion` (+ [zmienne CSS](./08-nowoczesny-css.md) dla motywów).
5. **Jak zbudować responsywną siatkę kart bez ani jednego media query?** — grid `auto-fill/minmax` lub flex wrap + basis.

## Dalsza lektura

- [web.dev: Learn Responsive Design](https://web.dev/learn/design)
- [MDN: Using container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)
- [Utopia — fluid type/space calculator](https://utopia.fyi/)
