# Nowoczesny CSS

> **Poziom:** 🟡 średni → 🔴 zaawansowany
> **Wymagana wiedza:** [Kaskada](./01-kaskada-specyficznosc.md), [Responsywność](./06-responsywnosc.md)

CSS ostatnich lat rozwiązał większość problemów, dla których powstały preprocesory i tony JS: zmienne, zagnieżdżanie, selektor rodzica, warstwy, container queries. Ten plik zbiera funkcje, które **są już baseline** (działają we wszystkich silnikach) i zmieniają sposób pisania stylów. (Container queries i `@layer` mają swoje domy: [responsywność](./06-responsywnosc.md), [kaskada](./01-kaskada-specyficznosc.md).)

## Custom properties (zmienne CSS)

```css
:root {
  --color-primary: oklch(55% 0.2 250);
  --space-md: 1rem;
}
.button {
  background: var(--color-primary);
  padding: var(--space-md);
  color: var(--text, white);          /* fallback */
}
```

Czym różnią się od zmiennych Sass ([preprocesory](./10-preprocesory-i-narzedzia.md)):
- **Żyją w runtime** — można je zmieniać per element, w media query, z JS (`el.style.setProperty('--x', '2rem')`).
- **Dziedziczą** i podlegają kaskadzie — nadpisanie na kontenerze przestyluje potomków. To silnik motywów (dark mode) i wariantów komponentów:

```css
.card { --accent: gray; border-color: var(--accent); }
.card--danger { --accent: red; }                     /* wariant = zmiana zmiennej */
[data-theme="dark"] { --color-bg: black; }           /* motyw = zestaw zmiennych */
```

- `@property` — rejestracja typu zmiennej (`syntax: '<color>'`), co umożliwia jej **animowanie** i waliduje wartości.

## Selektory, które zmieniają architekturę

- **`:has()` — „selektor rodzica"** i nie tylko:

```css
.field:has(input:invalid) { border-color: red; }     /* rodzic wg stanu dziecka */
body:has(dialog[open]) { overflow: hidden; }          /* blokada scrolla pod modalem */
.grid:has(> :nth-child(5)) { … }                      /* styl zależny od liczby dzieci */
```

- **`:is()` / `:where()`** — grupowanie selektorów (specificity: [kaskada](./01-kaskada-specyficznosc.md)).
- **Natywne zagnieżdżanie (nesting):**

```css
.card {
  color: black;
  &:hover { color: blue; }
  .title { font-weight: 700; }
  @media (min-width: 48rem) { padding: 2rem; }
}
```

- `:focus-visible` (focus tylko z klawiatury — [a11y](../10-dostepnosc/03-a11y-w-praktyce.md)), `:user-invalid`, `:nth-child(... of S)`.

## Funkcje i wartości

- `calc()` — miks jednostek: `width: calc(100% - 2rem)`; działa ze zmiennymi.
- `min()`, `max()`, `clamp()` — [jednostki](./02-box-model-i-jednostki.md) i [responsywność](./06-responsywnosc.md).
- **Kolory:** `oklch()` — percepcyjnie jednolita przestrzeń (ta sama „jasność" dla różnych barw — idealna do palet design systemu); `color-mix(in oklch, var(--c), white 20%)` — odcienie bez preprocesora; relative color syntax `oklch(from var(--c) calc(l - 0.1) c h)`.
- `light-dark(white, black)` — wartość zależna od motywu (z `color-scheme`).

## Nowe możliwości platformy (CSS + HTML)

- **`<dialog>`** — natywny modal: top layer (ponad wszystkimi [stacking contextami](./03-uklad-dokumentu.md)), focus trap, Esc, `::backdrop`.
- **Popover API** — `popovertarget`/`[popover]`: dropdowny/tooltipy bez JS, light dismiss, top layer. Do pozycjonowania: **anchor positioning** (przypinanie do elementu kotwicy; wsparcie rosnące).
- **View Transitions** — animowane przejścia między stanami/stronami (`document.startViewTransition()`, `view-transition-name`); w wariancie cross-document animują nawigacje MPA ([wzorce renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md)).
- **Scroll-driven animations** — [animacje](./07-animacje-i-transformacje.md).
- `content-visibility: auto` — przeglądarka pomija renderowanie treści poza ekranem ([wydajność](../09-wydajnosc/03-optymalizacja-dzialania.md)).
- `@scope` — ograniczanie zasięgu stylów do poddrzewa (scoping bez konwencji nazw; wsparcie rosnące — [architektura CSS](./09-architektura-css.md)).

## Jak śledzić wsparcie

- **Baseline** (widely available / newly available) — wspólny wskaźnik na MDN/caniuse.
- `@supports (display: grid) { … }` — feature queries: progressive enhancement zamiast wykrywania przeglądarek.

## Pułapki i częste błędy

- Zmienna CSS z literówką/niezdefiniowana = wartość invalid **at computed-value time** — właściwość dostaje initial/inherit, cicho i bez błędu; fallbacki `var(--x, fallback)` ratują.
- `:has()` jest kosztowny przy bardzo szerokich zasięgach (`body:has(...)` w ogromnym DOM) — używaj precyzyjnie.
- Traktowanie nowości jako „nieużywalnych" — połowa „nowego CSS" jest baseline od lat; sprawdzaj, zanim dopiszesz JS.
- Budowanie modala na divach w 2026 — `<dialog>` robi focus trap i top layer za darmo.
- Kolory w hex wszędzie — utrudnia motywy i warianty; zmienne + oklch/color-mix.

## Pytania rekrutacyjne

1. **Czym custom properties różnią się od zmiennych preprocesora?** — runtime, kaskada, dziedziczenie, dostęp z JS vs kompilacja.
2. **Podaj praktyczne zastosowania `:has()`.** — walidacja pól, stany rodzica, style zależne od zawartości/liczby dzieci.
3. **Jak dziś zaimplementujesz dark mode?** — `prefers-color-scheme` + zestawy zmiennych (+ `color-scheme`, `light-dark()`, przełącznik z `data-theme`).
4. **Co daje `<dialog>` względem własnego modala?** — top layer, focus management, Esc, `::backdrop`, semantyka.
5. **Jak bezpiecznie używać nowych funkcji CSS?** — Baseline, `@supports`, progressive enhancement.

## Dalsza lektura

- [MDN: CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties)
- [Chrome for Developers: New in CSS (coroczne podsumowania)](https://developer.chrome.com/blog)
- [web.dev: Baseline](https://web.dev/baseline)
