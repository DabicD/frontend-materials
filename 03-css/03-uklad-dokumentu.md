# Układ dokumentu: display, position, z-index

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Box model](./02-box-model-i-jednostki.md)

Zanim sięgniesz po [flexbox](./04-flexbox.md) i [grid](./05-grid.md), musisz rozumieć, jak elementy układają się „same z siebie" (normal flow), co robi `position` i jak naprawdę działa `z-index`. To wiedza, która odróżnia „stackowanie divów do skutku" od świadomego budowania layoutu.

## Normal flow i `display`

Bez żadnego CSS elementy płyną w **normal flow**:

- **block** — zajmuje całą dostępną szerokość, układa się pionowo (`div`, `p`, `section`).
- **inline** — płynie w linii tekstu; **ignoruje `width`/`height`**, pionowe marginesy nie odpychają linii (`span`, `a`, `strong`).
- **inline-block** — płynie w linii, ale respektuje wymiary; uwaga na „magiczne" odstępy z białych znaków w HTML.

`display` ma dziś składnię dwuczłonową: `display: block flex` = zewnętrzny typ (jak element zachowuje się wobec sąsiadów) + wewnętrzny (jak układa dzieci). Skróty `flex`, `grid`, `inline-flex`, `inline-grid`. Ponadto: `display: none` (usuwa z layoutu i accessibility tree) oraz `display: contents` (element „znika", dzieci wchodzą w jego miejsce — przydatne, ale historycznie problematyczne dla a11y).

## `position`

- **`static`** — domyślne; `top/left/z-index` nie działają.
- **`relative`** — element zostaje w flow (zajmuje swoje miejsce), ale można go przesunąć (`top/left…`) i **staje się punktem odniesienia** dla absolutnych potomków.
- **`absolute`** — wyjęty z flow; pozycjonowany względem najbliższego przodka z `position` ≠ `static` (technicznie: najbliższego **containing block** — tworzą go też `transform`, `filter`, `will-change`!). Brak takiego przodka → viewport (initial containing block).
- **`fixed`** — wyjęty z flow, przyklejony do viewportu. **Uwaga:** przodek z `transform`/`filter` przechwytuje fixed — element klei się do niego, nie do okna (częsty bug w modalach).
- **`sticky`** — hybryda: w flow, dopóki nie osiągnie progu (`top: 0`), potem „przykleja się" **w obrębie rodzica**. Wymaga podania progu; nie działa, gdy któryś przodek ma `overflow: hidden/auto` ze scrollowaniem, które go uwięzi.

Wzorzec „overlay wypełniający rodzica": rodzic `relative`, dziecko `absolute; inset: 0`.

## Stacking context i `z-index` — jak jest naprawdę

`z-index` **nie jest globalną liczbą**. Działa w obrębie **stacking contextu** — i porównywane są konteksty, nie pojedyncze liczby.

Stacking context tworzą m.in.: element root; `position` + `z-index` ≠ auto; `position: fixed/sticky`; `opacity < 1`; `transform`, `filter`, `perspective`, `clip-path`; `will-change`; `isolation: isolate`; `contain: layout/paint`; dziecko flex/grid z `z-index`.

**Konsekwencja:** element z `z-index: 9999` wewnątrz kontekstu o `z-index: 1` przegra z elementem `z-index: 2` na zewnątrz. Rozwiązania:
- świadome, płaskie zarządzanie warstwami (skala np. 10/20/30 w [zmiennych CSS](./08-nowoczesny-css.md)),
- `isolation: isolate` na komponencie — tworzy kontekst bez skutków ubocznych, „zamyka" wewnętrzne z-indexy,
- dla modali/tooltipów: renderowanie na końcu `<body>` (portal) i/lub natywny **`<dialog>`/Popover API z top layer** — warstwą ponad wszystkimi stacking contextami.

## Float i BFC — wiedza wciąż potrzebna

**Float** służy dziś do jednego: opływania obrazka tekstem. Historyczne layouty na floatach spotkasz w legacy.

**Block Formatting Context (BFC)** — „mini-świat layoutu": floaty nie wyciekają, marginesy się nie zlewają przez granicę, elementy nie opływają floatów z zewnątrz. Tworzą go m.in. `overflow` ≠ visible, `display: flow-root` (dedykowany, bez skutków ubocznych), flex/grid item, tabele.

```css
.container { display: flow-root; }  /* nowoczesny "clearfix" */
```

## Scrollowanie

- Kontener przewijalny = ograniczony rozmiar + `overflow: auto`.
- `scroll-behavior: smooth`, `scroll-margin-top` (kotwice pod sticky headerem), scroll snap (`scroll-snap-type`/`scroll-snap-align` — karuzele bez JS).
- `overscroll-behavior: contain` — blokuje „łańcuchowe" przewijanie strony pod modalem.
- Blokada scrolla body pod modalem: dziś po prostu natywny `<dialog>` + `body { overflow: hidden }` lub CSS `:has()` ([nowoczesny CSS](./08-nowoczesny-css.md)).

## Pułapki i częste błędy

- `top/left/z-index` „nie działają" — element jest `static`.
- Absolut pozycjonuje się „gdzieś w kosmosie" — brak `relative` na zamierzonym rodzicu.
- `position: fixed` łamie się w komponencie z animacją `transform` na przodku.
- Wojny `z-index: 999999` — zamiast tego zrozumienie kontekstów + `isolation` + top layer.
- `sticky` nie klei się — brak `top`, albo przodek z `overflow` tworzącym scroll container.
- `inline` element z ustawianym `width` — cicho ignorowane.

## Pytania rekrutacyjne

1. **Różnice między `relative`, `absolute`, `fixed`, `sticky`?** — flow/poza flow, punkt odniesienia, zachowanie przy scrollu.
2. **Względem czego pozycjonuje się `absolute`?** — najbliższy przodek tworzący containing block (`position`≠static, ale też transform/filter); inaczej viewport.
3. **Dlaczego element z wyższym `z-index` bywa pod elementem z niższym?** — porównują się stacking contexty rodziców, nie surowe liczby.
4. **Co tworzy stacking context?** — wymień kilka: opacity<1, transform, filter, fixed/sticky, z-index na positioned, isolation.
5. **Co to BFC i jak go dziś utworzyć bez skutków ubocznych?** — izolacja layoutu (floaty, collapsing); `display: flow-root`.

## Dalsza lektura

- [MDN: CSS positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout)
- [MDN: Stacking context](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Stacking_context)
- [web.dev: Learn CSS — Layout, Z-index and stacking contexts](https://web.dev/learn/css)
