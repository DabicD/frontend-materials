# Multimedia i grafika

> **Poziom:** 🟢 podstawowy → 🟡 średni (responsywne obrazy)
> **Wymagana wiedza:** [Struktura dokumentu](./01-struktura-dokumentu.md)

Obrazy to zwykle **największa część wagi strony** — a więc i największa dźwignia wydajności. Ten plik zbiera wiedzę o osadzaniu grafiki i wideo; aspekty czysto wydajnościowe (lazy loading, priorytety) rozwija [optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md).

## `<img>` — więcej niż `src`

```html
<img
  src="foto-800.jpg"
  alt="Górski szlak o wschodzie słońca"
  width="800" height="533"
  loading="lazy"
  decoding="async"
/>
```

- **`alt`** — tekst alternatywny: dla czytników ekranu, przy braku obrazu, dla robotów. Obraz czysto dekoracyjny → **`alt=""`** (pusty, ale obecny! wtedy czytnik go pomija). Opisuj funkcję, nie wygląd („Logo firmy — strona główna", nie „niebieski prostokąt").
- **`width`/`height`** — proporcje znane przed pobraniem = brak skoku layoutu (CLS — [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md)). CSS i tak może skalować (`max-width: 100%; height: auto`).
- **`loading="lazy"`** — natywny lazy loading poza viewportem. **Nigdy** dla obrazu hero/LCP.
- **`decoding="async"`** — dekodowanie poza main threadem.

## Responsywne obrazy: `srcset`, `sizes`, `<picture>`

**Problem 1: różne rozdzielczości/gęstości ekranu** → `srcset` + `sizes` (przeglądarka wybiera najlepszy plik):

```html
<img
  src="foto-800.jpg"
  srcset="foto-400.jpg 400w, foto-800.jpg 800w, foto-1600.jpg 1600w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="…"
/>
```

`400w` = szerokość pliku w pikselach; `sizes` mówi, ile miejsca obraz zajmie w layoucie — z tego i z DPR przeglądarka liczy, który plik pobrać.

**Problem 2: inny kadr/format zależnie od warunków (art direction)** → `<picture>`:

```html
<picture>
  <source type="image/avif" srcset="foto.avif" />
  <source type="image/webp" srcset="foto.webp" />
  <source media="(max-width: 600px)" srcset="foto-kadr-pionowy.jpg" />
  <img src="foto.jpg" alt="…" />  <!-- fallback; to on niesie alt/width/height -->
</picture>
```

**Formaty (od najlepszej kompresji):** AVIF → WebP → JPEG (foto) / PNG (przezroczystość, zrzuty) / GIF (nie używaj — wideo lub WebP animowany). SVG dla grafiki wektorowej.

## SVG vs Canvas

| | SVG | Canvas |
|---|---|---|
| Model | deklaratywny, elementy w DOM | imperatywny, piksele na bitmapie |
| Skalowanie | bezstratne | rozmycie przy skalowaniu |
| Interakcje | eventy per element, stylowanie CSS | ręczna detekcja trafień |
| Wydajność | spada przy tysiącach elementów | świetna przy dużej liczbie obiektów (gry, wykresy z 100k punktów) |
| Dostępność | tekst czytelny dla maszyn | czarna skrzynka (wymaga fallbacków) |

SVG można osadzić przez `<img src="ikona.svg">` (izolowany, cache'owalny) albo **inline** `<svg>…</svg>` (stylowalny CSS-em, np. `fill: currentColor` — standard dla ikon). Do grafiki 3D/gier: WebGL/WebGPU (poza zakresem kompendium).

## `<video>` i `<audio>`

```html
<video controls poster="okladka.jpg" preload="metadata" width="1280" height="720">
  <source src="film.webm" type="video/webm" />
  <source src="film.mp4" type="video/mp4" />
  <track kind="captions" src="napisy-pl.vtt" srclang="pl" label="Polski" />
</video>
```

- Atrybuty: `controls`, `autoplay` (działa tylko z `muted`! polityka przeglądarek), `loop`, `poster`, `preload` (`none`/`metadata`/`auto`).
- **`<track>`** — napisy WebVTT; wymóg dostępności dla treści mówionych ([podstawy a11y](../10-dostepnosc/01-podstawy-a11y.md)).
- „GIF-owe" animacje: `<video muted autoplay loop playsinline>` — wielokrotnie mniejsze niż GIF.
- Streaming adaptacyjny (HLS/DASH) wymaga JS (np. hls.js) — natywnie tylko Safari.

## Pułapki i częste błędy

- Brak `width`/`height` (lub `aspect-ratio` w CSS) — skoki layoutu podczas ładowania.
- `loading="lazy"` na obrazie hero — pogarsza LCP zamiast poprawiać.
- Brak `alt` w ogóle (czytnik czyta nazwę pliku: „IMG-dwadzieścia-trzy-final-v2…") albo `alt` „obrazek".
- Serwowanie PNG 4000px do miniatury 200px — brak `srcset`/resize'owania.
- `autoplay` bez `muted` — „czemu wideo nie startuje?".
- Ikony jako font ikon — problemy a11y i FOUT; dziś standardem jest inline SVG.

## Pytania rekrutacyjne

1. **Jak działa `srcset`/`sizes` i czym różni się od `<picture>`?** — wybór rozdzielczości przez przeglądarkę vs jawna dyrektywa (format/kadr).
2. **Kiedy SVG, a kiedy Canvas?** — wektorowa grafika/ikony/interakcje w DOM vs masowe rysowanie pikseli.
3. **Jak zapobiec przesunięciom layoutu przy ładowaniu obrazów?** — `width`/`height` lub `aspect-ratio`, placeholdery.
4. **Dlaczego `autoplay` nie działa?** — polityki autoplay: wymagane `muted` lub wcześniejsza interakcja użytkownika.
5. **Jak napisać dobry `alt`?** — funkcja obrazu w kontekście; dekoracyjne → `alt=""`.

## Dalsza lektura

- [MDN: Responsive images](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Responsive_images)
- [web.dev: Learn Images](https://web.dev/learn/images)
- [MDN: Video and audio content](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/HTML_video_and_audio)
