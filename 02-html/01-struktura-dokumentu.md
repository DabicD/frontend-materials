# Struktura dokumentu HTML

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md) (pomocna, nie wymagana)

HTML to szkielet każdej strony — nawet najbardziej „JS-owej" SPA. Poprawna struktura dokumentu wpływa na renderowanie, SEO, dostępność i wydajność ładowania. Ten plik omawia elementy „niewidoczne": doctype, `<head>` i strategie ładowania zasobów.

## Minimalny poprawny dokument

```html
<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Tytuł strony</title>
</head>
<body>
  <!-- treść -->
</body>
</html>
```

- **`<!DOCTYPE html>`** — nie jest tagiem HTML, tylko instrukcją: „renderuj w **standards mode**". Bez niego przeglądarka wchodzi w **quirks mode** — emulację błędów z lat 90. (np. inny box model), co psuje layout w subtelny sposób.
- **`lang="pl"`** — język dokumentu: wpływa na czytniki ekranu (wymowa!), dzielenie wyrazów, tłumaczenie automatyczne i SEO.
- **`<meta charset="UTF-8">`** — deklaracja kodowania; musi być w pierwszych 1024 bajtach. Bez niej ryzyko „krzaczków" (mojibake).
- **`<meta name="viewport" ...>`** — bez tego mobile renderuje stronę w wirtualnym oknie ~980px i skaluje w dół; fundament [responsywności](../03-css/06-responsywnosc.md).

## Co żyje w `<head>`

- `<title>` — tytuł karty, wynik wyszukiwania, zakładki; kluczowy dla [SEO](./05-seo-i-metadane.md).
- `<meta name="description">`, Open Graph itd. — dom tematu: [SEO i metadane](./05-seo-i-metadane.md).
- `<link rel="stylesheet">` — CSS (render-blocking — zob. [jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)).
- `<link rel="icon">`, `rel="manifest"` (PWA — [szczegóły](../08-architektura-aplikacji/02-spa-mpa-pwa.md)), `rel="canonical"`.
- Resource hints: `preload`, `prefetch`, `preconnect`, `dns-prefetch` — dom: [optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md).
- `<base href>` — bazowy URL dla linków względnych (rzadko używany, potrafi zaskoczyć).

## Ładowanie skryptów: `async`, `defer`, `type="module"`

Klasyczny `<script src>` **zatrzymuje parser HTML** do czasu pobrania i wykonania. Alternatywy:

```html
<script src="a.js"></script>                <!-- blokuje parser -->
<script async src="b.js"></script>         <!-- pobiera równolegle, wykonuje NATYCHMIAST po pobraniu -->
<script defer src="c.js"></script>         <!-- pobiera równolegle, wykonuje po sparsowaniu HTML, przed DOMContentLoaded -->
<script type="module" src="d.js"></script> <!-- ESM: domyślnie zachowuje się jak defer -->
```

| | Pobieranie | Wykonanie | Kolejność między skryptami |
|---|---|---|---|
| (brak) | blokuje parser | natychmiast | zachowana |
| `async` | równoległe | po pobraniu (przerywa parser) | **losowa** (kto pierwszy) |
| `defer` | równoległe | po HTML, przed `DOMContentLoaded` | zachowana |
| `module` | równoległe (z zależnościami) | jak `defer` | wg grafu importów |

**Praktyka:** `defer` (lub `type="module"`) dla kodu aplikacji; `async` tylko dla skryptów niezależnych (analityka). Skrypt na końcu `<body>` to historyczny odpowiednik `defer` — dziś niepotrzebny.

**Zdarzenia cyklu życia dokumentu:**
- `DOMContentLoaded` — HTML sparsowany, DOM gotowy (obrazki mogą się jeszcze ładować).
- `load` (na `window`) — wszystkie zasoby (obrazy, style) pobrane.

## Kategorie treści i zasady zagnieżdżania

HTML ma reguły, czego walidator (i parser!) pilnuje:

- Elementy blokowe vs inline to koncepcja CSS; w HTML5 mówimy o kategoriach: **flow content**, **phrasing content**, **interactive content**…
- Klasyczne błędy zagnieżdżania, które parser **naprawia po swojemu** (rozjeżdżając DOM z kodem źródłowym): `<p>` w `<p>`, `<div>` w `<p>`, `<button>` w `<button>`, `<a>` w `<a>`, `<li>` bezpośrednio w `<div>`.
- Formularze nie mogą być zagnieżdżane w sobie.

Dlatego „HTML działa" ≠ „HTML jest poprawny" — parser jest liberalny, ale wynikowy DOM może różnić się od intencji (co psuje CSS-owe selektory i hydrację SSR — [wzorce renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md)).

## Pułapki i częste błędy

- Brak `<meta name="viewport">` — strona „działa", ale na telefonie jest mikroskopijna.
- `async` dla skryptów zależnych od siebie — race condition objawiający się losowo.
- Manipulowanie DOM przed `DOMContentLoaded` bez `defer` — `document.querySelector` zwraca `null`.
- `lang="en"` na polskiej stronie (kopiuj-wklej z boilerplate) — czytniki ekranu czytają polski tekst angielską fonetyką.
- Zagnieżdżenie interaktywnego w interaktywnym (`<button>` w `<a>`) — nieprzewidywalne zachowanie i problem dostępności.

## Pytania rekrutacyjne

1. **Czym różni się `async` od `defer`?** — oba pobierają równolegle; `async` wykonuje od razu po pobraniu (bez gwarancji kolejności), `defer` po sparsowaniu HTML (z kolejnością).
2. **Po co jest `<!DOCTYPE html>`?** — wymusza standards mode; bez niego quirks mode i niespójny rendering.
3. **Różnica między `DOMContentLoaded` a `load`?** — DOM gotowy vs wszystkie zasoby pobrane.
4. **Dlaczego umieszczano skrypty przed `</body>`?** — by nie blokować parsowania; dziś zastępuje to `defer`/`type="module"`.
5. **Co się stanie, gdy zagnieździsz `<div>` w `<p>`?** — parser zamknie `<p>` przed `<div>` — DOM będzie inny niż kod źródłowy.

## Dalsza lektura

- [MDN: Document and website structure](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content)
- [MDN: `<script>` — async/defer](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script)
- [HTML Living Standard](https://html.spec.whatwg.org/) — źródło prawdy
