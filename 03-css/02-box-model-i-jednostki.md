# Box model i jednostki

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Kaskada i specificity](./01-kaskada-specyficznosc.md)

Każdy element na stronie jest prostokątnym pudełkiem. Box model definiuje, z czego to pudełko się składa i jak liczony jest jego rozmiar — a jednostki decydują, w czym te rozmiary wyrażasz. Brzmi trywialnie, ale to tu rodzą się „tajemnicze" rozjazdy layoutu.

## Box model

Od środka: **content → padding → border → margin**.

```
┌─ margin (przezroczysty, odpycha sąsiadów)
│ ┌─ border
│ │ ┌─ padding (tło elementu obejmuje padding)
│ │ │ ┌─ content (width × height)
```

**`box-sizing`** decyduje, czego dotyczy `width`/`height`:

- `content-box` (domyślne) — `width` = sam content; padding i border **dokładają się** do rozmiaru.
- `border-box` — `width` = content + padding + border (intuicyjne: „pudełko ma 300px i koniec").

Standardowy reset (używaj zawsze):

```css
*, *::before, *::after { box-sizing: border-box; }
```

## Margin — trzy niespodzianki

1. **Margin collapsing** — pionowe marginesy sąsiadów (i rodzica z pierwszym/ostatnim dzieckiem!) **zlewają się** do większego z nich, zamiast się sumować. Dotyczy tylko normal flow — nie działa we [flex/grid](./04-flexbox.md), przy `overflow` innym niż `visible`, floatach i elementach absolutnych. Klasyczny objaw: margines dziecka „wycieka" nad rodzica.
2. **Marginesy procentowe** (także `margin-top`!) liczą się od **szerokości** rodzica.
3. `margin: auto` w poziomie centruje element blokowy o ustalonej szerokości (a we flex/grid działa w obu osiach — najprostsze centrowanie ikony w przycisku).

## Jednostki

**Absolutne:** `px` (piksel CSS — nie fizyczny piksel ekranu; przelicznik to `devicePixelRatio`).

**Względne względem czcionki:**
- `em` — względem `font-size` **elementu** (dla `font-size`: rodzica). Kaskaduje/mnoży się przy zagnieżdżeniu — bywa pułapką.
- `rem` — względem `font-size` **roota** (`html`, domyślnie 16px). Przewidywalny; **standard dla typografii i spacingu**.
- `ch` (szerokość „0" — świetne do `max-width` tekstu: `max-width: 65ch`), `ex`, `lh` (line-height).

**Względem viewportu:** `vw`, `vh` (1% szerokości/wysokości okna), `vmin`, `vmax` oraz warianty dynamiczne dla mobile: `svh`/`lvh`/`dvh` (małe/duże/dynamiczne — rozwiązują problem znikającego paska adresu; `100dvh` zamiast `100vh`).

**Względem kontenera** (przy container queries): `cqw`, `cqh`, `cqi`, `cqb` — [responsywność](./06-responsywnosc.md).

**Procenty** — zawsze pytaj: „procent CZEGO?". `width: 50%` — szerokości rodzica; `height: 50%` — wysokości rodzica **o ile ta jest określona** (inaczej ignorowane); `transform: translateX(50%)` — rozmiaru **samego elementu**.

**Dlaczego `rem`, nie `px`, dla tekstu:** użytkownik może zmienić bazowy rozmiar czcionki w ustawieniach przeglądarki (nie zoom!) — `px` to ignoruje, `rem` skaluje. To wymóg [dostępności](../10-dostepnosc/03-a11y-w-praktyce.md).

## `min-/max-width`, funkcje rozmiaru

- `min-width`/`max-width` nadpisują `width` (min wygrywa z max przy konflikcie).
- `min()`, `max()`, `clamp(min, preferowana, max)`:

```css
width: min(90%, 1200px);            /* "mniejsze z dwóch" — kontener strony */
font-size: clamp(1rem, 2.5vw, 1.5rem);  /* fluid typography — patrz responsywność */
```

- Słowa kluczowe: `min-content` (najwęższy bez overflow), `max-content` (naturalna szerokość bez łamania), `fit-content`.
- `aspect-ratio: 16 / 9` — proporcje bez hacka z `padding-top`.

## `overflow` i rozmiar zawartości

`overflow: visible | hidden | scroll | auto | clip`. Wartość inna niż `visible` tworzy nowy **block formatting context** (BFC) i **containing block dla layoutu** — co m.in. wyłącza margin collapsing i zatrzymuje floaty ([układ dokumentu](./03-uklad-dokumentu.md)). `overflow-x`/`-y` można ustawiać osobno. Pamiętaj: „tajemniczy poziomy scrollbar" to zwykle element szerszy niż viewport — znajdź go w DevTools.

## Logical properties

Zamiast kierunków fizycznych (`left/right/top/bottom`) — kierunki logiczne, zależne od kierunku pisma (`direction`, `writing-mode`):

- `margin-inline-start` (≈ left w LTR, right w RTL), `padding-block`, `inset-inline`, `border-start-start-radius`…
- `inline` = oś tekstu, `block` = oś bloków.
- Skróty dwuwartościowe: `margin-inline: auto` (centrowanie), `padding-block: 1rem`.

Jeśli produkt kiedykolwiek dostanie wersję RTL (arabski, hebrajski) — logical properties zamieniają „przepisz pół CSS-a" na „zero pracy". Warto pisać tak od dziś.

## Pułapki i częste błędy

- Brak resetu `box-sizing: border-box` — element z `width: 100%` i paddingiem wystaje z rodzica.
- Margin collapsing traktowany jak bug — to spec; kontroluj przez padding rodzica, `gap` we flex/grid, albo BFC.
- `height: 100%` „nie działa" — rodzic nie ma określonej wysokości; częsty łańcuszek `html, body { height: 100% }` albo po prostu `min-height: 100dvh`.
- `100vh` na mobile — pasek adresu zasłania dół; używaj `dvh`/`svh`.
- Zagnieżdżone `em` (np. `font-size: 0.9em` w komponencie w komponencie) — tekst maleje wykładniczo.

## Pytania rekrutacyjne

1. **`content-box` vs `border-box`?** — czego dotyczy zadeklarowany `width`; czemu border-box jest standardem.
2. **Co to jest margin collapsing i kiedy NIE występuje?** — zlewanie pionowych marginesów w normal flow; nie we flex/grid/BFC/floatach/absolutach.
3. **`em` vs `rem` — różnica i kiedy które?** — element/rodzic vs root; rem do typografii i spacingu, em do rzeczy skalujących się z lokalnym fontem (np. padding przycisku).
4. **Dlaczego `height: 100%` często nie działa, a `width: 100%` tak?** — procent wysokości wymaga zdefiniowanej wysokości rodzica.
5. **Co robią logical properties i po co?** — kierunki zależne od trybu pisania; internacjonalizacja RTL bez przepisywania stylów.

## Dalsza lektura

- [MDN: The box model](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Box_model)
- [MDN: CSS values and units](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Values_and_Units)
- [web.dev: Learn CSS — Box model, Sizing, Logical properties](https://web.dev/learn/css)
