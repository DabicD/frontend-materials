# Semantyka HTML

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Struktura dokumentu](./01-struktura-dokumentu.md)

Semantyka = używanie znaczników zgodnie z ich **znaczeniem**, nie wyglądem. Wygląd i tak nadpisze CSS; znaczenie konsumują czytniki ekranu, wyszukiwarki, tryby czytania i inne maszyny. Semantyczny HTML to darmowa [dostępność](../10-dostepnosc/01-podstawy-a11y.md) i [SEO](./05-seo-i-metadane.md) — a jego brak to najczęstszy grzech „divowego" frontendu.

## Landmarki — mapa strony

```html
<body>
  <header>logo, nawigacja główna</header>
  <nav>menu</nav>
  <main>            <!-- dokładnie JEDEN na stronę -->
    <article>samodzielna treść (np. post, produkt)</article>
    <aside>treść poboczna (sidebar, powiązane)</aside>
  </main>
  <footer>stopka</footer>
</body>
```

Landmarki pozwalają użytkownikom czytników ekranu **skakać po regionach** strony zamiast słuchać jej liniowo. `article` vs `section` vs `div`:

- **`<article>`** — treść samodzielna, ma sens wyjęta z kontekstu (post, komentarz, karta produktu, widget RSS).
- **`<section>`** — tematyczna sekcja dokumentu, zwykle z własnym nagłówkiem; użyj, gdy sekcja „zasługuje na miejsce w spisie treści".
- **`<div>`** — brak znaczenia; **wyłącznie** hak na style/layout. Używanie diva nie jest grzechem — grzechem jest div tam, gdzie istnieje element semantyczny.

## Nagłówki `h1–h6`

- Tworzą **konspekt dokumentu** — jak spis treści książki. Użytkownicy czytników nawigują po nagłówkach częściej niż po czymkolwiek innym.
- **Nie przeskakuj poziomów** (`h1` → `h3`) i **nie dobieraj nagłówka rozmiarem czcionki** — rozmiar to sprawa CSS.
- Jeden `h1` na stronę (temat główny) to nadal najbezpieczniejsza praktyka.

## Semantyka tekstu i interakcji

- `<strong>` (ważne) / `<em>` (akcent) zamiast `<b>`/`<i>` — te drugie są czysto prezentacyjne.
- `<time datetime="2026-07-06">`, `<figure>` + `<figcaption>`, `<blockquote>`, `<code>`, `<kbd>`, `<mark>`.
- Listy: `<ul>`/`<ol>`/`<dl>` — czytnik zapowiada „lista, 5 elementów"; menu nawigacyjne to `<nav><ul>…`.
- Tabele **do danych tabelarycznych** (z `<th scope>`, `<caption>`), nigdy do layoutu.

**Najważniejsza para do zapamiętania:**

- **`<a href>`** — nawigacja (zmiana URL). Działa z Ctrl+klik, „otwórz w nowej karcie", jest widoczna dla robotów.
- **`<button>`** — akcja (submit, otwarcie modala, toggle). Domyślnie focusowalny, obsługuje Enter i Spację.

`<div onClick>` zamiast któregokolwiek z nich = element niewidoczny dla klawiatury i czytnika ekranu; naprawa wymaga ręcznie `tabindex`, roli i obsługi klawiszy ([ARIA i semantyka](../10-dostepnosc/02-aria-i-semantyka.md)) — czyli odtworzenia za darmo dostępnej funkcjonalności.

## Semantyka a maszyny

1. **Czytniki ekranu** budują z semantyki drzewo dostępności (accessibility tree) — [szczegóły](../10-dostepnosc/02-aria-i-semantyka.md).
2. **Wyszukiwarki** lepiej rozumieją strukturę treści ([SEO](./05-seo-i-metadane.md)).
3. **Tryb czytania** przeglądarek wyciąga `<article>`/`<main>`.
4. **Przyszłe AI/scrapery** — dobrze otagowana treść jest po prostu czytelniejsza maszynowo.

## Pułapki i częste błędy

- „Div soup" — wszystko divami, interakcje na `onClick`; kosztem dostępności i SEO.
- `<section>` jako zamiennik `<div>` „bo brzmi profesjonalniej" — sekcja bez nagłówka to zwykle zły wybór.
- Wiele `<main>` lub `<h1>` w wygenerowanych częściach strony (częste przy komponentach składanych z różnych źródeł).
- Nagłówki dobierane rozmiarem („h4 bo mniejszy") — psuje konspekt.
- Ikona-button bez tekstu i bez `aria-label` — czytnik ogłasza „przycisk" bez nazwy.

## Pytania rekrutacyjne

1. **Po co semantyka, skoro CSS i tak nadaje wygląd?** — znaczenie dla maszyn: dostępność, SEO, tryby czytania; wygląd ≠ znaczenie.
2. **`<article>` vs `<section>` vs `<div>`?** — samodzielna treść / tematyczna sekcja z nagłówkiem / brak znaczenia.
3. **Kiedy `<a>`, a kiedy `<button>`?** — nawigacja (URL) vs akcja; różnice w obsłudze klawiatury i zachowaniu przeglądarki.
4. **Co daje poprawna hierarchia nagłówków?** — nawigację czytnikiem ekranu i zrozumiały konspekt dla wyszukiwarek.
5. **Jakie problemy tworzy `<div onClick>`?** — brak focusa, brak obsługi klawiatury, brak roli — niewidzialny dla technologii asystujących.

## Dalsza lektura

- [MDN: HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements)
- [web.dev: Learn HTML — Semantic HTML](https://web.dev/learn/html/semantic-html)
- [HTML5 Doctor — archiwum wyjaśnień elementów](http://html5doctor.com/)
