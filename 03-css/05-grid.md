# CSS Grid

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Flexbox](./04-flexbox.md)

Grid to layout **dwuwymiarowy**: definiujesz siatkę wierszy i kolumn, a potem umieszczasz w niej elementy — kontrola nad oboma wymiarami naraz. To narzędzie do layoutów stron, dashboardów, galerii i formularzy. Reguła kciuka: **grid = layout projektowany „od kontenera", flexbox = rozmieszczanie „od treści"**.

## Definiowanie siatki

```css
.layout {
  display: grid;
  grid-template-columns: 280px 1fr 1fr;   /* 3 kolumny */
  grid-template-rows: auto 1fr auto;      /* nagłówek, treść, stopka */
  gap: 1rem 2rem;                         /* row-gap column-gap */
}
```

- **`fr`** — ułamek wolnej przestrzeni (po odjęciu rozmiarów stałych i gap).
- **`repeat()`** — `repeat(3, 1fr)`, `repeat(auto-fill, minmax(250px, 1fr))`.
- **`minmax(min, max)`** — ograniczenia toru.
- Rozmiary od treści: `auto`, `min-content`, `max-content`, `fit-content(400px)`.

**Najważniejszy wzorzec responsywny (bez media queries!):**

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1rem;
}
```

`auto-fill` tworzy tyle kolumn ≥250px, ile się zmieści; `auto-fit` dodatkowo **zwija puste** tory (nieliczne elementy rozciągają się na całość).

## Umieszczanie elementów

**Po liniach** (linie numerują się od 1; ujemne od końca):

```css
.item  { grid-column: 1 / 3; }        /* od linii 1 do 3 = 2 kolumny */
.item2 { grid-column: span 2; }       /* zajmij 2 tory */
.full  { grid-column: 1 / -1; }       /* cała szerokość */
```

**Po nazwanych obszarach** — czytelne jak ASCII-art:

```css
.layout {
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 280px 1fr;
}
header { grid-area: header; }
```

**Auto-placement:** elementy bez jawnej pozycji płyną wg `grid-auto-flow: row | column | row dense` (`dense` wypełnia dziury — uwaga na rozjazd kolejności wizualnej z DOM). Wiersze tworzone automatycznie wymiaruje `grid-auto-rows`.

## Wyrównanie

Ta sama rodzina co we flexbox: `justify-*` = oś pozioma (inline), `align-*` = pionowa (block):

- `justify-items` / `align-items` — treść **w komórkach** (domyślnie `stretch`).
- `justify-content` / `align-content` — **cała siatka** w kontenerze (gdy siatka mniejsza od kontenera).
- `justify-self` / `align-self` — pojedynczy element.
- `place-items: center` — skrót; `display: grid; place-items: center` to najkrótsze centrowanie w CSS.

## Subgrid i zagnieżdżanie

Dziecko grida może samo być gridem (zagnieżdżenie — osobne siatki). **`subgrid`** pozwala dziecku **dziedziczyć tory rodzica**:

```css
.card { grid-row: span 3; display: grid; grid-template-rows: subgrid; }
```

Klasyczny use case: karty w siatce, w których nagłówki/opisy/stopki **wyrównują się między kartami** niezależnie od długości treści. Wsparcie: wszystkie silniki (baseline od 2023).

## Grid vs Flexbox — decyzja

| Sytuacja | Narzędzie |
|---|---|
| Layout strony/sekcji (obszary) | Grid |
| Galeria kafelków o równych wymiarach | Grid (`auto-fill/minmax`) |
| Pasek: logo + menu + przycisk | Flexbox |
| Elementy „tyle, ile mają treści" + zawijanie | Flexbox + wrap |
| Wyrównanie wewnątrz komponentu (ikona+tekst) | Flexbox |
| Nakładanie elementów na siebie (hero z tekstem na obrazie) | Grid (elementy w tej samej komórce) zamiast `absolute` |

Nakładanie w gridzie: dwa elementy z `grid-area: 1 / 1` — responsywne i bez wyjmowania z flow.

## Pułapki i częste błędy

- `auto-fill` vs `auto-fit` — mylone; różnica widoczna, gdy elementów mało.
- `fr` nie działa „przez” min-content: duży element (obrazek, `<pre>`) rozpycha kolumnę `1fr` — analogicznie do flexboxa pomaga `minmax(0, 1fr)` lub `min-width: 0`.
- Wysokość „na oko": `height: 100%` w komórkach zamiast pozwolić gridowi wymiarować (`1fr`, `align-items: stretch`).
- Nazwane obszary muszą tworzyć prostokąty — literówka w `grid-template-areas` unieważnia całą definicję.
- Grid „nie działa" na dzieciach dzieci — gridem sterujesz tylko bezpośrednie dzieci (chyba że `subgrid`/`display: contents`).

## Pytania rekrutacyjne

1. **Grid czy flexbox — jak decydujesz?** — wymiary layoutu (2D vs 1D), projekt od kontenera vs od treści.
2. **Jak działa `repeat(auto-fill, minmax(250px, 1fr))`?** — automatyczna liczba kolumn, minimalna szerokość, elastyczne wypełnianie; różnica z `auto-fit`.
3. **Co daje `subgrid`?** — współdzielenie torów rodzica; wyrównanie treści między rodzeństwem (karty).
4. **Jak nałożyć elementy na siebie bez `position: absolute`?** — ta sama komórka grida (`grid-area: 1/1`).
5. **Czym jest `fr` i czym różni się od `%`?** — ułamek wolnej przestrzeni (po gap/stałych) vs procent całości kontenera.

## Dalsza lektura

- [CSS-Tricks: A Complete Guide to CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [MDN: CSS grid layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout)
- [Grid Garden](https://cssgridgarden.com/) — ćwiczenia interaktywne
