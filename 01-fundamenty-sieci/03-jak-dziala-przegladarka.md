# Jak działa przeglądarka — od HTML do pikseli

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Jak działa internet](./01-jak-dziala-internet.md), [Protokół HTTP](./02-protokol-http.md)

Przeglądarka to najważniejsze środowisko uruchomieniowe frontend developera. Zrozumienie, jak zamienia bajty HTML/CSS/JS na piksele (tzw. **rendering pipeline** / **critical rendering path**), tłumaczy niemal wszystkie zasady wydajności: czemu CSS „blokuje", czemu `async`/`defer` istnieją i czemu animowanie `width` jest złe, a `transform` dobre.

## Architektura przeglądarki

- **Silnik renderujący (rendering engine):** Blink (Chrome, Edge, Opera), WebKit (Safari), Gecko (Firefox) — parsuje HTML/CSS i maluje piksele.
- **Silnik JavaScript:** V8 (Chrome/Edge/Node), JavaScriptCore (Safari), SpiderMonkey (Firefox) — parsuje i wykonuje JS (z kompilacją JIT).
- **Architektura wieloprocesowa:** osobne procesy na przeglądarkę, GPU, sieć i (zwykle) każdą kartę/origin — awaria jednej karty nie ubija reszty, a izolacja procesów wzmacnia bezpieczeństwo (site isolation).
- W obrębie karty działa **main thread** — jeden wątek wykonujący JS, style, layout i część malowania. To wąskie gardło całego frontendu; jego kolejkowanie opisuje [event loop](../04-javascript/09-event-loop.md).

## Critical Rendering Path — krok po kroku

```
HTML ──parsowanie──▶ DOM ──┐
                           ├─▶ Render tree ─▶ Layout ─▶ Paint ─▶ Composite
CSS ──parsowanie──▶ CSSOM ─┘
```

1. **Parsowanie HTML → DOM.** Strumieniowe (przeglądarka nie czeka na cały plik). Napotkany `<script>` bez atrybutów **zatrzymuje parser** — JS może przecież zmienić dokument (`document.write`). Dlatego skrypty dostały atrybuty `async`/`defer` ([szczegóły](../02-html/01-struktura-dokumentu.md)).
2. **Parsowanie CSS → CSSOM.** CSS jest **render-blocking**: przeglądarka nie wyrenderuje nic, dopóki nie zna stylów (uniknięcie FOUC — flash of unstyled content). Co więcej, wykonanie JS czeka na CSSOM (skrypt może pytać o style), więc wolny CSS blokuje też JS.
3. **Render tree** = DOM + CSSOM, tylko elementy widoczne (`display: none` wypada; `visibility: hidden` zostaje — zajmuje miejsce).
4. **Layout (reflow)** — obliczenie geometrii: pozycji i rozmiarów każdego boxu (algorytmy: [box model](../03-css/02-box-model-i-jednostki.md), [flexbox](../03-css/04-flexbox.md), [grid](../03-css/05-grid.md)).
5. **Paint** — rasteryzacja: zamiana boxów na piksele (tła, teksty, cienie) w obrębie warstw.
6. **Composite** — złożenie warstw (layers) w finalny obraz, na **wątku kompozytora** z pomocą GPU. Właściwości `transform` i `opacity` mogą być animowane na tym etapie **z pominięciem layoutu i paintu** — stąd ich wydajność ([animacje](../03-css/07-animacje-i-transformacje.md), [optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)).

**Preload scanner:** równolegle z głównym parserem lekki skaner szuka w HTML zasobów do pobrania z wyprzedzeniem (`img`, `link`, `script`) — dlatego zasoby wpisane wprost w HTML startują szybciej niż odkrywane przez JS/CSS.

## Reflow i repaint w trakcie życia strony

Po pierwszym renderze każda zmiana ma swój koszt:

- zmiana geometrii (`width`, `font-size`, dodanie elementu) → **layout + paint + composite** (najdroższa),
- zmiana wyglądu bez geometrii (`background-color`) → **paint + composite**,
- `transform` / `opacity` → często **tylko composite** (najtańsza).

Odczyt właściwości geometrycznych (`offsetHeight`, `getBoundingClientRect()`) **wymusza synchroniczny layout**, jeśli są niezaaplikowane zmiany — przeplatanie zapisów i odczytów w pętli to **layout thrashing** (dom tematu: [optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)).

## Klatka renderowania (frame)

Przy 60 Hz przeglądarka ma ~16,7 ms na klatkę: `input → rAF callbacks → style → layout → paint → composite`. Jeśli main thread jest zajęty JS-em dłużej, klatki są gubione — to widoczny **jank**. Stąd zasada dzielenia długich tasków i API `requestAnimationFrame` ([przydatne API](../05-dom-i-web-api/07-przydatne-api-przegladarki.md)).

## Pułapki i częste błędy

- `<script>` w `<head>` bez `defer`/`async` — blokuje parsowanie i opóźnia pierwszy render.
- Wielki, jeden plik CSS jako render-blocking dla całej strony — warto wydzielić krytyczny CSS ([optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md)).
- Animowanie `top/left/width/height` zamiast `transform` — wymusza layout w każdej klatce.
- Mylenie `display: none` (brak w render tree, brak layoutu) z `visibility: hidden` (jest w layoucie) i `opacity: 0` (jest i w layoucie, i w paincie, łapie eventy).
- Zakładanie, że „przeglądarki działają tak samo" — silniki różnią się szczegółami; testuj przynajmniej w Blink i WebKit (mobile Safari!).

## Pytania rekrutacyjne

1. **Opisz critical rendering path.** — DOM + CSSOM → render tree → layout → paint → composite; CSS render-blocking, JS parser-blocking.
2. **Dlaczego CSS blokuje renderowanie, a JS parsowanie?** — bez stylów byłby FOUC; JS może modyfikować dokument i odpytywać style, więc czeka i na parser, i na CSSOM.
3. **Różnica między reflow a repaint? Co je wyzwala?** — reflow = geometria (droższy), repaint = piksele; wyzwalacze: zmiany stylów, odczyty geometrii, resize.
4. **Dlaczego `transform: translateX()` jest wydajniejsze niż `left`?** — omija layout i paint, wykonuje się na kompozytorze/GPU.
5. **Co to jest layout thrashing?** — naprzemienne zapisy i odczyty geometrii wymuszające wielokrotny synchroniczny layout; rozwiązanie: batchowanie odczytów/zapisów.

## Dalsza lektura

- [web.dev: Critical rendering path](https://web.dev/learn/performance/understanding-the-critical-path)
- [Inside look at modern web browser (seria, Chrome team)](https://developer.chrome.com/blog/inside-browser-part1)
- [CSS Triggers — która właściwość wyzwala layout/paint/composite](https://csstriggers.com/)
