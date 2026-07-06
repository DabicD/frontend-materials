# ARIA i drzewo dostępności

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Podstawy a11y](./01-podstawy-a11y.md), [Semantyka HTML](../02-html/02-semantyka.md)

ARIA (Accessible Rich Internet Applications) to zestaw atrybutów uzupełniających semantykę tam, gdzie HTML nie wystarcza. **Pierwsza zasada ARIA: nie używaj ARIA** — jeśli istnieje natywny element, użyj jego. ARIA nie dodaje zachowań (focus, klawiatura) — tylko **opisuje** element maszynom; obietnice bez pokrycia czynią rzeczy gorszymi.

## Drzewo dostępności (accessibility tree)

Przeglądarka buduje z DOM równoległą strukturę dla technologii asystujących. Każdy węzeł ma:

- **role** — czym jest (`button`, `navigation`, `dialog`, `listbox`),
- **name** (accessible name) — jak się nazywa (etykieta ogłaszana przez czytnik),
- **state/properties** — stan (`expanded`, `checked`, `disabled`, `selected`),
- **value** — np. wartość slidera.

Elementy natywne dostają wszystko za darmo: `<button>Zapisz</button>` → role=button, name="Zapisz", stany focus/disabled + obsługa klawiatury. `<div onClick>` → **nic** ([semantyka](../02-html/02-semantyka.md)).

**Obliczanie accessible name (musisz znać priorytety):** `aria-labelledby` → `aria-label` → natywne (label formularza / `alt` / treść tekstowa / `title` jako ostatnia deska). Uwaga: `aria-label` **nadpisuje** treść widoczną (button z tekstem „Kup teraz" i `aria-label="przycisk"` ogłosi „przycisk" — bug).

## Role, właściwości i stany

```html
<!-- role — TYLKO gdy brak natywnego elementu -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel-1" id="tab-1">Opis</button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">…</div>

<!-- właściwości: relacje i opisy -->
<input aria-describedby="pwd-hint" aria-invalid="true" />
<p id="pwd-hint">Min. 12 znaków.</p>

<!-- stany: aktualizowane z JS! -->
<button aria-expanded="false" aria-controls="menu">Menu</button>
```

- **`aria-label` / `aria-labelledby`** — nazwa (własna / wskazanie elementu); **`aria-describedby`** — dodatkowy opis (hinty, błędy).
- **Stany:** `aria-expanded` (rozwinięte?), `aria-selected`, `aria-checked`, `aria-current="page"` (aktywny link nawigacji), `aria-disabled`, `aria-busy`, `aria-pressed` (toggle button).
- **`aria-hidden="true"`** — usuń z drzewa dostępności (ikony dekoracyjne, zduplikowana treść). **Nigdy** na elemencie z focusowalnym środkiem.
- Rzeczy ukryte poprawnie: `display:none`/`hidden` znika wszędzie; `aria-hidden` — tylko dla czytników; klasa `.visually-hidden` (klip 1px) — **tylko wizualnie**, czytnik czyta (etykiety dla ikon, nagłówki sekcji).

## Live regions — ogłaszanie zmian

Czytnik nie widzi, że „coś się pojawiło" — dynamiczne komunikaty wymagają live region:

```html
<div aria-live="polite" class="visually-hidden" id="status"></div>
<script>status.textContent = 'Dodano do koszyka';</script>
```

- `polite` — po zakończeniu bieżącej wypowiedzi (większość przypadków: toasty, wyniki wyszukiwania, statusy zapisu).
- `assertive` — przerwij natychmiast (tylko krytyczne błędy).
- Role skrótowe: `role="status"` (=polite), `role="alert"` (=assertive).
- Kluczowe detale: region musi **istnieć w DOM zanim** wstawisz komunikat; ogłaszana jest **zmiana** treści.
- Zastosowania: wyniki filtrowania („12 produktów"), walidacja async, postęp, [nawigacja SPA](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md) (ogłoszenie nowej strony).

## Wzorce widgetów (APG)

**ARIA Authoring Practices Guide (APG)** — oficjalny katalog wzorców: jakie role, stany **i jaka klawiatura** obowiązują dla: accordion, combobox, dialog, menu, tabs, tooltip, treegrid… Zanim zbudujesz własny widget:

1. Sprawdź, czy nie ma natywnego (`<dialog>`, `<details>`, `<select>`, Popover — [nowoczesny CSS](../03-css/08-nowoczesny-css.md)).
2. Sprawdź APG — wzorzec definiuje kontrakt (np. tabs: strzałki między zakładkami, Tab wychodzi do panelu).
3. Rozważ gotową bibliotekę **headless** (Radix, Headless UI, React Aria) — dostępność zaimplementowana, wygląd Twój.

Ręczne pisanie comboboxa zgodnego z APG to tygodnie — decyzja „budujemy sami" ma być świadoma.

## Pułapki i częste błędy

- ARIA bez zachowań: `role="button"` bez `tabindex="0"` i obsługi Enter/Spacji — czytnik ogłasza przycisk, który nie działa.
- Redundantne role: `<button role="button">`, `<nav role="navigation">`.
- `aria-label` na elementach nieinteraktywnych (div/span bez roli) — często ignorowane.
- Stany ustawione raz i nieaktualizowane (`aria-expanded` na sztywno `false`).
- `aria-hidden="true"` na kontenerze z aktywnym focusem (np. tło pod modalem razem z modalem).
- Live region tworzony dynamicznie razem z komunikatem — cisza.
- Ikony-fonty/SVG bez `aria-hidden` — czytnik czyta śmieci.

## Pytania rekrutacyjne

1. **Czym jest accessibility tree i skąd się bierze?** — DOM+semantyka+ARIA → role/name/state.
2. **Pierwsza zasada ARIA?** — preferuj natywny HTML; ARIA opisuje, nie implementuje.
3. **Jak jest wyliczany accessible name?** — labelledby → label → natywne mechanizmy; pułapka nadpisania.
4. **Jak ogłosisz czytnikowi wynik akcji asynchronicznej?** — live region polite/assertive; wymagania (istnieje wcześniej).
5. **Zbuduj dostępne tabs — co trzeba obsłużyć?** — role tablist/tab/tabpanel, aria-selected, strzałki + Tab (APG), powiązania id.

## Dalsza lektura

- [ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/patterns/) — katalog wzorców, absolutna podstawa
- [MDN: WAI-ARIA basics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/WAI-ARIA_basics)
- [Sara Soueidan: blog o a11y](https://www.sarasoueidan.com/blog/)
