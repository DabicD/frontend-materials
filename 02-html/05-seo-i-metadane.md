# SEO i metadane

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Struktura dokumentu](./01-struktura-dokumentu.md), [Semantyka](./02-semantyka.md)

SEO (Search Engine Optimization) z perspektywy frontend developera to: dostarczenie robotom **indeksowalnej treści**, **metadanych** opisujących strony i **szybkiego, poprawnego renderowania**. Nie zajmujemy się tu marketingową stroną SEO (dobór słów kluczowych) — tylko warstwą techniczną, za którą odpowiada kod.

## Jak wyszukiwarka widzi stronę

1. **Crawling** — robot pobiera HTML i podąża za linkami `<a href>` (JS-owe „linki" bez `href` są niewidoczne).
2. **Rendering** — Google wykonuje JS (Chromium), ale w **drugiej fali**, z opóźnieniem i budżetem; inne roboty (i scrapery, podglądy social media) często JS **nie wykonują**.
3. **Indexing** — treść trafia do indeksu.

Wniosek architektoniczny: treść, która ma być indeksowana, powinna być w HTML z serwera — stąd [SSR/SSG](../08-architektura-aplikacji/01-wzorce-renderowania.md) dla stron publicznych. Czysty CSR (SPA) bywa wystarczający dla Google, ale ryzykowny dla reszty świata.

## Podstawowe metadane w `<head>`

```html
<title>Buty trailowe X | Sklep Górski</title>
<meta name="description" content="Zwięzły opis strony na ok. 150 znaków — to potencjalny snippet w wynikach." />
<link rel="canonical" href="https://example.com/buty/x" />
<meta name="robots" content="index, follow" />  <!-- lub noindex, nofollow -->
```

- **`<title>`** — najważniejszy sygnał on-page; unikalny per podstrona, temat na początku.
- **`description`** — nie wpływa na ranking wprost, wpływa na CTR (to często Twój snippet).
- **`canonical`** — wskazuje wersję „prawdziwą" przy duplikatach (parametry URL, wersje z/bez `www`, paginacja).
- **`robots`** — `noindex` dla stron, które nie powinny być w Google (panel, wyniki wyszukiwania wewnętrznego, strony testowe!).

## Open Graph i karty społecznościowe

Podglądy linków (Facebook/LinkedIn/Slack/komunikatory) czytają **Open Graph**; X/Twitter — Twitter Cards:

```html
<meta property="og:title" content="Buty trailowe X" />
<meta property="og:description" content="…" />
<meta property="og:image" content="https://example.com/og/buty-x.jpg" />  <!-- absolutny URL, ~1200x630 -->
<meta property="og:type" content="product" />
<meta property="og:url" content="https://example.com/buty/x" />
<meta name="twitter:card" content="summary_large_image" />
```

Te tagi **muszą być w HTML z serwera** — boty podglądów nie wykonują JS. To najczęstszy praktyczny argument za SSR/SSG nawet w „aplikacyjnych" projektach.

## Structured data (Schema.org / JSON-LD)

Dane strukturalne pozwalają na **rich results** (gwiazdki, ceny, FAQ, breadcrumbs w wynikach):

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Buty trailowe X",
  "offers": { "@type": "Offer", "price": "349.00", "priceCurrency": "PLN" }
}
</script>
```

JSON-LD (rekomendowany przez Google) nie dubluje treści widocznej — opisuje ją maszynowo. Testowanie: Google Rich Results Test.

## Pozostałe elementy techniczne

- **`robots.txt`** — instrukcje crawlowania (czego nie odwiedzać); **nie ukrywa** treści (to nie mechanizm bezpieczeństwa).
- **`sitemap.xml`** — lista URL-i do zaindeksowania; generowana automatycznie przy buildzie.
- **Semantyka i nagłówki** — [semantyka](./02-semantyka.md) to sygnały struktury treści.
- **Wydajność** — Core Web Vitals są czynnikiem rankingowym ([szczegóły](../09-wydajnosc/01-core-web-vitals.md)).
- **Mobile-first indexing** — Google indeksuje wersję mobilną; [responsywność](../03-css/06-responsywnosc.md) to warunek wejściowy.
- **Statusy HTTP mają znaczenie:** 404 dla nieistniejących, 301 dla przeniesionych ([protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md)); „soft 404" (strona błędu ze statusem 200) myli roboty.
- **hreflang** — wersje językowe: `<link rel="alternate" hreflang="pl" href="…" />`.

## Pułapki i częste błędy

- SPA bez SSR: puste `<div id="root"></div>` w HTML — brak podglądów social media, ryzyko słabej indeksacji poza Google.
- Ten sam `<title>`/`description` na wszystkich podstronach (typowe w SPA bez zarządzania head).
- `noindex` z środowiska stagingowego wypchnięty na produkcję (lub odwrotnie — staging zaindeksowany, bo bez `noindex`).
- OG image ze ścieżką względną albo generowany przez JS — podgląd linku pusty.
- Nawigacja na `onClick` + `history.pushState` bez prawdziwych `<a href>` — robot nie znajdzie podstron.
- Traktowanie `robots.txt` jak kontroli dostępu.

## Pytania rekrutacyjne

1. **Jak SPA wpływa na SEO i jak to naprawić?** — rendering JS w drugiej fali / brak wykonania JS przez część botów; SSR/SSG/prerendering dla treści publicznych.
2. **Do czego służy `rel="canonical"`?** — deduplikacja: wskazanie preferowanego URL dla tej samej treści.
3. **Dlaczego tagi OG muszą być renderowane serwerowo?** — boty podglądów nie wykonują JS.
4. **Czym jest JSON-LD?** — maszynowy opis treści (Schema.org) umożliwiający rich results.
5. **Jak wydajność wiąże się z SEO?** — Core Web Vitals jako czynnik rankingowy + wpływ na crawl budget i konwersję.

## Dalsza lektura

- [Google Search Central: JavaScript SEO basics](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics)
- [The Open Graph protocol](https://ogp.me/)
- [Schema.org](https://schema.org/) + [Google: Structured data](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data)
