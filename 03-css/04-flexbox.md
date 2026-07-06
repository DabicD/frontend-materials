# Flexbox

> **Poziom:** 🟢 podstawowy → 🟡 średni (algorytm rozmiarów)
> **Wymagana wiedza:** [Układ dokumentu](./03-uklad-dokumentu.md)

Flexbox to layout **jednowymiarowy** — rozmieszcza elementy wzdłuż jednej osi (wiersz LUB kolumna) i rozdziela między nie wolną przestrzeń. Idealny do: pasków nawigacji, wyrównywania w komponentach, centrowania, rozpychania elementów. Do siatek dwuwymiarowych — [grid](./05-grid.md).

## Model myślowy: dwie osie

`display: flex` na kontenerze definiuje:
- **main axis** — kierunek z `flex-direction` (`row` domyślnie),
- **cross axis** — prostopadła.

Właściwości wyrównania działają względem osi, nie ekranu — `justify-*` = main axis, `align-*` = cross axis. Przy `flex-direction: column` „justify" działa pionowo!

## Kontener

```css
.container {
  display: flex;
  flex-direction: row | column;      /* (+ -reverse) */
  flex-wrap: nowrap | wrap;          /* czy łamać do nowych linii */
  justify-content: flex-start | center | space-between | space-around | space-evenly;
  align-items: stretch | center | flex-start | flex-end | baseline;
  align-content: …;                  /* rozkład CAŁYCH linii — tylko przy wrap */
  gap: 1rem;                         /* odstępy — zamiast marginesów! */
}
```

- `align-items: stretch` (domyślne) — elementy rozciągają się na wysokość linii (stąd „równe kolumny za darmo").
- `baseline` — wyrównanie linii bazowej tekstu (przydatne: label + input różnych rozmiarów).
- `gap` działa we flex od lat — koniec z hackami `margin` + ujemny margines na kontenerze.

## Elementy: `flex-grow`, `flex-shrink`, `flex-basis`

Serce flexboxa — algorytm dzielenia przestrzeni:

- **`flex-basis`** — rozmiar startowy elementu na main axis (domyślnie `auto` = z contentu/width).
- **`flex-grow`** — ile **nadwyżki** przestrzeni element dostaje (proporcja; 0 = nie rośnie).
- **`flex-shrink`** — jak bardzo się kurczy przy **niedoborze** (domyślnie 1 = kurczy się; 0 = nie).

```css
.item { flex: 1; }          /* = 1 1 0% — równe kolumny (basis 0: dzielimy CAŁOŚĆ) */
.item { flex: auto; }       /* = 1 1 auto — rośnie od naturalnego rozmiaru */
.item { flex: none; }       /* = 0 0 auto — sztywny */
.sidebar { flex: 0 0 280px; }  /* stała szerokość, treść obok: flex: 1 */
```

**Kluczowa subtelność — `flex: 1` vs `flex-grow: 1`:** przy `basis: 0%` elementy dostają równe szerokości niezależnie od treści; przy `basis: auto` dzielona jest tylko nadwyżka, więc element z dłuższą treścią będzie szerszy.

**`min-width: auto` — źródło 80% frustracji:** flex item domyślnie **nie skurczy się poniżej rozmiaru swojej treści** (min-content). Dlatego długi tekst/`<pre>`/obrazek „rozpycha" layout mimo `flex-shrink`. Lek:

```css
.item { min-width: 0; }   /* pozwól się kurczyć; analogicznie min-height: 0 w kolumnie */
```

(typowo razem z `overflow: hidden; text-overflow: ellipsis; white-space: nowrap` dla przycinania tekstu).

## Pojedyncze elementy: `align-self`, `order`, `margin: auto`

- `align-self` — nadpisanie `align-items` dla jednego elementu.
- `order` — zmiana kolejności wizualnej. **Ostrożnie:** kolejność DOM (czytniki, Tab) się nie zmienia — ryzyko dostępnościowe ([a11y w praktyce](../10-dostepnosc/03-a11y-w-praktyce.md)).
- `margin-left: auto` na elemencie — „zjada" całą wolną przestrzeń: najprostszy sposób na „wszystko w lewo, ostatni element w prawo".

## Klasyczne wzorce

```css
/* Centrowanie totalne */
.parent { display: flex; justify-content: center; align-items: center; }

/* Navbar: logo — menu — akcje */
.nav { display: flex; align-items: center; gap: 2rem; }
.nav .actions { margin-left: auto; }

/* Sticky footer strony */
body { min-height: 100dvh; display: flex; flex-direction: column; }
main { flex: 1; }

/* Karty w wierszach, min 250px, dopychane */
.cards { display: flex; flex-wrap: wrap; gap: 1rem; }
.card { flex: 1 1 250px; }
```

## Pułapki i częste błędy

- `min-width: auto` (opisane wyżej) — element nie chce się kurczyć.
- Mylenie osi przy `flex-direction: column` — `justify-content` działa wtedy pionowo.
- `width` na flex itemach zamiast `flex-basis`/`flex` — działa, ale walczy z algorytmem; bądź konsekwentny.
- Odstępy marginesami zamiast `gap` — marginesy skrajnych elementów i łamanie linii robią bałagan.
- `flex-wrap` zapomniane — treść wystaje z kontenera na wąskich ekranach.
- Nadużywanie `order` — rozjazd kolejności wizualnej i DOM.

## Pytania rekrutacyjne

1. **Kiedy flexbox, a kiedy grid?** — 1D (dystrybucja wzdłuż osi, rozmiar od treści) vs 2D (siatka, rozmiar od layoutu); szczegóły także w [pliku o grid](./05-grid.md).
2. **Co dokładnie znaczy `flex: 1`?** — `1 1 0%`; wyjaśnij rolę basis=0 dla równych kolumn.
3. **Czemu flex item nie kurczy się mimo `flex-shrink: 1`?** — domyślne `min-width: auto` = min-content; naprawa `min-width: 0`.
4. **Jak działa `margin: auto` we flexboxie?** — pochłania wolną przestrzeń na danej osi; wzorzec push-right/centrowanie.
5. **Różnica `align-items` vs `align-content`?** — elementy w linii vs całe linie (tylko multi-line).

## Dalsza lektura

- [CSS-Tricks: A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) — klasyka, ściąga
- [MDN: Flexbox](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Flexbox)
- [Flexbox Froggy](https://flexboxfroggy.com/) — ćwiczenia interaktywne
