# Preprocesory i narzędzia CSS

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Nowoczesny CSS](./08-nowoczesny-css.md) (żeby wiedzieć, czego już NIE potrzebujesz od preprocesora)

Preprocesory (Sass/SCSS, Less, Stylus) rozszerzają CSS o mechanizmy programistyczne i kompilują się do zwykłego CSS w buildzie. Ich rola **zmalała** — zmienne, zagnieżdżanie i funkcje matematyczne są już natywne — ale Sass wciąż spotkasz w większości starszych i wielu nowych projektów, a PostCSS napędza niemal każdy pipeline (często niejawnie).

## Sass/SCSS — co trzeba znać

```scss
@use 'sass:color';
@use './tokens' as t;                 // system modułów (@use/@forward, NIE stary @import)

$radius: 8px;                          // zmienna KOMPILACYJNA (vs runtime'owe custom properties)

@mixin focus-ring($color: t.$primary) {   // mixin — parametryzowany blok
  outline: 2px solid $color;
  outline-offset: 2px;
}

.card {
  border-radius: $radius;
  &:hover { box-shadow: 0 2px 8px rgb(0 0 0 / 0.15); }   // & = selektor rodzica
  &__title { font-weight: 700; }                          // BEM przez &
  @include focus-ring;
}

// pętle/warunki — generowanie utilities
@each $name, $size in (sm: 0.5rem, md: 1rem, lg: 2rem) {
  .p-#{$name} { padding: $size; }
}
```

- **`@use`/`@forward`** zastąpiły `@import` (deprecated: globalny namespace, wielokrotne wklejanie).
- **`@extend`** — dziedziczenie selektorów; używaj oszczędnie (generuje zaskakujące łańcuchy selektorów), zwykle mixin jest bezpieczniejszy.
- Funkcje wbudowane (`color.adjust`, `math.div`) i własne (`@function`).
- **Partials** (`_plik.scss`) — struktura wg [ITCSS](./09-architektura-css.md).

**Sass vs natywny CSS — decyzja 2026:** zmienne → natywne custom properties (runtime!); zagnieżdżanie → natywne; `calc()` → natywny. Sass zostaje silny w: **generowaniu kodu** (pętle, mapy → utilities), mixinach, dzieleniu na moduły w projektach bez bundlera. Nowe projekty często nie potrzebują Sass wcale.

## PostCSS — inna kategoria narzędzia

PostCSS to **platforma transformacji CSS przez wtyczki** (parsuje CSS do AST, wtyczki go modyfikują). Nie konkuruje z Sass — często działa PO nim. Kluczowe wtyczki:

- **Autoprefixer** — dodaje prefiksy vendorowe wg danych caniuse i Twojego `browserslist`. Standard w każdym buildzie.
- **postcss-preset-env** — pozwala pisać przyszły CSS, transpiluje do wspieranego (odpowiednik Babela dla CSS).
- **cssnano** — minifikacja.
- Na PostCSS zbudowane są: Tailwind (silnik), CSS Modules (implementacje), lintery (Stylelint).

**`browserslist`** (w `package.json` lub `.browserslistrc`) — wspólna deklaracja wspieranych przeglądarek dla Autoprefixera, Babela i innych narzędzi:

```
> 0.5%, last 2 versions, not dead
```

## Miejsce w pipeline

```
SCSS ──(sass)──▶ CSS ──(PostCSS: autoprefixer…)──▶ CSS ──(minifikacja)──▶ dist/
```

W praktyce spina to bundler ([bundlery i build](../14-narzedzia/03-bundlery-i-proces-budowania.md)) — np. Vite ma PostCSS i obsługę `.scss` (po doinstalowaniu sass) out of the box. Lintowanie: **Stylelint** ([jakość kodu](../14-narzedzia/04-jakosc-kodu.md)).

## Pułapki i częste błędy

- Zmienne Sass tam, gdzie potrzebny runtime (motywy, dark mode) — po kompilacji są „wypieczone"; do themingu tylko [custom properties](./08-nowoczesny-css.md).
- Głębokie zagnieżdżanie w Sass (`.a { .b { .c { … }}}`) — produkuje silne selektory sprzężone z DOM ([specificity](./01-kaskada-specyficznosc.md)); zasada: max 1–2 poziomy, głównie `&`.
- Stary `@import` w nowym kodzie.
- Ręczne pisanie prefiksów `-webkit-` — od tego jest Autoprefixer + browserslist.
- Przestarzały `browserslist` w projekcie — build celuje w przeglądarki sprzed dekady i puchnie.

## Pytania rekrutacyjne

1. **Czy w nowym projekcie użyjesz Sass? Uzasadnij.** — świadomość, co jest już natywne; Sass dla codegen/mixinów, często zbędny.
2. **Zmienne Sass vs custom properties?** — czas kompilacji vs runtime; kaskada, dziedziczenie, JS, motywy.
3. **Czym jest PostCSS i czym różni się od preprocesora?** — platforma wtyczek na AST; transformacje, nie nowy język.
4. **Jak działa Autoprefixer?** — dane caniuse + browserslist → automatyczne prefiksy.
5. **Dlaczego głębokie zagnieżdżanie to antywzorzec?** — specificity, sprzężenie z DOM, trudne nadpisywanie.

## Dalsza lektura

- [Sass — dokumentacja](https://sass-lang.com/documentation/)
- [PostCSS](https://postcss.org/)
- [Browserslist](https://browsersl.ist/)
