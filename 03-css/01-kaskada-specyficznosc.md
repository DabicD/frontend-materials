# Kaskada, specificity i dziedziczenie

> **Poziom:** 🟢 podstawowy (a jednak najczęściej niezrozumiany)
> **Wymagana wiedza:** [Struktura dokumentu HTML](../02-html/01-struktura-dokumentu.md)

„Dlaczego ten styl nie działa?!" — w 90% przypadków odpowiedzią jest kaskada lub specificity. To mechanizm rozstrzygania konfliktów: gdy wiele reguł celuje w ten sam element i właściwość, **kaskada** wybiera zwycięzcę. Zrozumienie jej raz na zawsze kończy programowanie CSS metodą „dopisz `!important` i module".

## Algorytm kaskady — kolejność rozstrzygania

Dla każdej właściwości elementu przeglądarka porównuje deklaracje w tej kolejności (pierwsze rozstrzygnięcie wygrywa):

1. **Pochodzenie i ważność:** style przeglądarki (user-agent) < style użytkownika < **style autora** < style autora `!important` < user `!important` < UA `!important`. (Uwaga: `!important` **odwraca** kolejność pochodzeń).
2. **Warstwy kaskady (`@layer`)** — niżej.
3. **Specificity** — niżej.
4. **Kolejność w źródle** — późniejsza deklaracja wygrywa.

## Specificity — trzy liczby (A, B, C)

Dla selektora liczysz trójkę `(A, B, C)` i porównujesz leksykograficznie (A najważniejsze):

- **A** — liczba selektorów **ID** (`#header`)
- **B** — liczba **klas** (`.btn`), **atrybutów** (`[type="text"]`), **pseudoklas** (`:hover`)
- **C** — liczba **typów** (`div`) i **pseudoelementów** (`::before`)

```css
p                    /* (0,0,1) */
.card p              /* (0,1,1) */
#main .card p:hover  /* (1,2,1) */
style=""             /* atrybut inline bije wszystko poza !important */
```

Reguły specjalne:
- `*`, kombinatory (`>`, `+`, `~`) — nie liczą się.
- `:where(...)` — **zawsze (0,0,0)** (świetne do stylów bazowych, które łatwo nadpisać).
- `:is(...)`, `:has(...)`, `:not(...)` — przyjmują specificity **najsilniejszego argumentu**.
- Specificity **nie przechodzi między liczbami**: 11 klas `(0,11,0)` nadal przegrywa z 1 ID `(1,0,0)`.

**Praktyka senior-level:** utrzymuj specificity **płaską i niską** — najlepiej pojedyncze klasy `(0,1,0)`. To rdzeń metodologii typu BEM ([architektura CSS](./09-architektura-css.md)). Wysokie specificity to dług: każde nadpisanie wymaga jeszcze wyższego.

## `!important`

Wygrywa ze wszystkim w swoim pochodzeniu — i dlatego jest ostatecznością. Uzasadnione użycia: style utility (`.hidden { display: none !important }`), nadpisywanie styli third-party, których nie kontrolujesz. Jeśli potrzebujesz go we własnym kodzie — masz problem architektoniczny, nie CSS-owy.

## `@layer` — warstwy kaskady

Nowoczesne rozwiązanie wojen specificity: **kolejność warstw bije specificity** wewnątrz pochodzenia autora.

```css
@layer reset, base, components, utilities;  /* deklaracja kolejności */

@layer base {
  #main a { color: blue; }        /* (1,0,1), ale warstwa base... */
}
@layer utilities {
  .link-red { color: red; }       /* ...przegrywa z utilities — .link-red WYGRYWA */
}
```

Style **poza warstwami** wygrywają z warstwowymi (unlayered > layered). Typowe zastosowanie: reset i style frameworka w niskich warstwach, własny kod wyżej — koniec z `!important` do nadpisywania bibliotek.

## Dziedziczenie

Niektóre właściwości **dziedziczą** z rodzica: głównie typografia (`color`, `font-*`, `line-height`, `text-align`) i `visibility`. Nie dziedziczą: layout i box model (`width`, `margin`, `padding`, `border`, `display`, `position`).

Wartości sterujące (działają na każdej właściwości):
- `inherit` — weź od rodzica (wymuś dziedziczenie).
- `initial` — wartość początkowa ze specyfikacji (uwaga: dla `display` to `inline`!).
- `unset` — `inherit` jeśli właściwość dziedziczy, inaczej `initial`.
- `revert` — cofnij do stylu przeglądarki (user-agent).

Wartość wygrana w kaskadzie → jeśli brak, dziedziczenie → jeśli brak, wartość initial. Custom properties (`--zmienna`) dziedziczą — [nowoczesny CSS](./08-nowoczesny-css.md).

## Pułapki i częste błędy

- Naprawianie specificity kolejnym, dłuższym selektorem — spirala `#app .page .card.card a.link` → refaktor do płaskich klas albo `@layer`.
- `:where()` vs `:is()` — identyczna składnia, inna specificity; do resetów zawsze `:where()`.
- Selektory generowane przez preprocesor (`&` zagnieżdżenia w [Sass](./10-preprocesory-i-narzedzia.md)) — łatwo nieświadomie wyprodukować głębokie, silne selektory.
- Zapominanie, że style inline bije tylko `!important`, a `!important` autora bije user-agentowe, ale nie userowe `!important` (dostępność: użytkownik ma ostatnie słowo).
- Debugowanie na ślepo — DevTools pokazuje przekreślone deklaracje i **dlaczego** przegrały ([DevTools](../14-narzedzia/05-devtools.md)).

## Pytania rekrutacyjne

1. **Jak liczy się specificity?** — (ID, klasy/atrybuty/pseudoklasy, typy/pseudoelementy), porównanie leksykograficzne; inline i `!important` ponad tym.
2. **Który selektor wygra: `#nav a` czy `.nav .link.active`?** — `#nav a` = (1,0,1) bije (0,3,0)? Nie! (1,0,1) > (0,3,0), bo A rozstrzyga. Wygrywa `#nav a`.
3. **Czym różni się `:is()` od `:where()`?** — specificity argumentu vs zawsze zero.
4. **Do czego służy `@layer`?** — jawna kolejność warstw ważniejsza niż specificity; porządkuje style frameworków/resetów.
5. **Które właściwości dziedziczą i jak wymusić/wyłączyć dziedziczenie?** — typografia dziedziczy; `inherit`/`initial`/`unset`/`revert`.

## Dalsza lektura

- [MDN: Cascade, specificity, and inheritance](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Handling_conflicts)
- [Specificity calculator (interaktywny)](https://specificity.keegan.st/)
- [CSS Cascade — wizualne wyjaśnienie (wattenberger)](https://2019.wattenberger.com/blog/css-cascade)
