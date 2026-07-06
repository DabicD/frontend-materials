# Architektura CSS

> **Poziom:** 🟡 średni → 🔴 zaawansowany (decyzje zespołowe)
> **Wymagana wiedza:** [Kaskada i specificity](./01-kaskada-specyficznosc.md), [Nowoczesny CSS](./08-nowoczesny-css.md)

CSS jest globalny, kaskadowy i bez enkapsulacji — w dużym projekcie bez konwencji zamienia się w „append-only stylesheet", gdzie nikt nie usuwa reguł, bo nie wiadomo, co się zepsuje. Architektura CSS to zestaw strategii **izolacji** (żeby style nie wyciekały) i **przewidywalności** (żeby wiadomo było, gdzie co jest). To temat o kompromisach — na rozmowach senior pada jako „jak zorganizujesz style w dużym projekcie?".

## Problem do rozwiązania

1. **Globalny namespace** — każda klasa może kolidować z każdą.
2. **Specificity wars** — nadpisania wymagają coraz silniejszych selektorów.
3. **Martwy kod** — brak pewności, czy selektor jest jeszcze używany.
4. **Zależność od struktury DOM** — selektory potomków (`.card div span`) pękają przy refaktorze HTML.

## Metodologie konwencji nazw

**BEM (Block — Element — Modifier)** — najbardziej znana:

```css
.card { }                /* blok — samodzielny komponent */
.card__title { }         /* element — część bloku */
.card--featured { }      /* modyfikator — wariant */
```

Zasady: wszystko klasami (płaska specificity (0,1,0)), zero zależności od DOM (`.card__title`, nie `.card h2`), elementy nie zagnieżdżają się w nazwach (`.card__header__title` ❌). Wady: gadatliwość, dyscyplina ręczna.

**ITCSS (Inverted Triangle)** — organizacja **warstw** od ogólnych do szczegółowych: settings → tools → generic (reset) → elements → objects → components → utilities. Specificity rośnie w dół trójkąta. Dziś naturalnie mapuje się na natywne [`@layer`](./01-kaskada-specyficznosc.md). Podejścia się łączą: ITCSS jako struktura plików/warstw + BEM jako nazewnictwo komponentów.

## Izolacja przez narzędzia

**CSS Modules** — pliki `*.module.css`; klasy są lokalne, bundler generuje unikalne nazwy:

```css
/* Button.module.css */ .primary { background: navy; }
```
```js
import styles from './Button.module.css';
element.className = styles.primary;    // "Button_primary__x7f2a"
```

Zalety: prawdziwa izolacja, zwykły CSS, zero runtime. To dziś **domyślny bezpieczny wybór** w projektach komponentowych.

**CSS-in-JS** (styled-components, Emotion) — style w JS, kolokacja z komponentem, dynamiczne style z props. Koszt: runtime (serializacja, wstrzykiwanie — problem przy SSR i wydajności), sprzężenie z frameworkiem. Nowa fala to **zero-runtime CSS-in-JS** (vanilla-extract, Linaria, StyleX) — składnia JS/TS, ekstrakcja do statycznego CSS w buildzie.

**Utility-first (Tailwind CSS)** — komponowanie z małych klas jednozadaniowych:

```html
<button class="px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-700">…</button>
```

Zalety: zero wymyślania nazw, ograniczony design system (skale odstępów/kolorów), lokalność zmian (edytujesz HTML, nie globalny CSS), świetne usuwanie nieużywanych stylów. Wady: „długi HTML", logika wariantów przenosi się do templatek, wymaga tooling buy-in. **Duplikacji klas nie zwalcza się przez `@apply`, tylko przez komponenty** (to framework komponentowy jest warstwą abstrakcji).

**Shadow DOM** — izolacja na poziomie platformy (Web Components): style nie wchodzą i nie wychodzą. Mocna enkapsulacja, ale utrudnia theming i współpracę z globalnym design systemem; głównie w widgetach/design systemach dystrybuowanych szeroko.

## Jak wybrać? (heurystyki)

| Kontekst | Sensowny wybór |
|---|---|
| Aplikacja komponentowa (SPA/SSR) | CSS Modules albo Tailwind |
| Zespół z silnym design systemem i tokenami | Tailwind (config = tokeny) lub Modules + [zmienne CSS](./08-nowoczesny-css.md) |
| Strona treściowa / mało komponentów | zwykły CSS + ITCSS/@layer + BEM |
| Biblioteka komponentów dystrybuowana do obcych aplikacji | zero-runtime, zmienne CSS jako API themingu; ew. Shadow DOM |
| Legacy z wojnami specificity | `@layer` + stopniowa migracja, `:where()` dla resetów |

Niezależnie od wyboru: **design tokens jako custom properties** (`--color-*`, `--space-*`) są wspólnym mianownikiem wszystkich podejść.

## Pułapki i częste błędy

- Mieszanie trzech podejść naraz bez decyzji — najgorsza z opcji.
- Głębokie selektory potomków sprzężone z DOM — pękają przy każdej zmianie struktury.
- `@apply` w Tailwind jako sposób na „posprzątanie HTML" — odtwarza problemy klasycznego CSS; komponenty są odpowiedzią.
- CSS-in-JS z runtime w aplikacji wrażliwej na wydajność/SSR — mierz koszt hydracji.
- Brak strategii usuwania martwego CSS — coverage w [DevTools](../14-narzedzia/05-devtools.md), lintery, izolacja per komponent.

## Pytania rekrutacyjne

1. **Jak zorganizujesz CSS w dużym, wieloosobowym projekcie?** — konwencja (BEM/ITCSS) albo izolacja narzędziowa (Modules/Tailwind/zero-runtime) + tokeny + @layer; uzasadnij kontekstem.
2. **Jakie problemy rozwiązuje BEM i jakim kosztem?** — płaska specificity, brak kolizji; gadatliwość, dyscyplina.
3. **CSS Modules vs CSS-in-JS vs Tailwind — kompromisy?** — izolacja/runtime/DX/wydajność; umiej wskazać kontekst dla każdego.
4. **Czym są design tokens?** — nazwane decyzje projektowe (kolory, spacing, typografia) jako zmienne — jedno źródło prawdy dla stylów.
5. **Jak ugryźć legacy CSS z wysoką specificity?** — audyt, `@layer`, `:where()`, stopniowa migracja komponentowa ([praca z legacy](../15-inzynieria-oprogramowania/03-praca-z-legacy.md)).

## Dalsza lektura

- [BEM — oficjalna metodologia](https://en.bem.info/methodology/)
- [ITCSS — wprowadzenie (xfive)](https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture/)
- [Tailwind CSS: Utility-first fundamentals](https://tailwindcss.com/docs/styling-with-utility-classes)
- [CSS Modules — dokumentacja](https://github.com/css-modules/css-modules)
