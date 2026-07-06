# Web Workers — wątki w przeglądarce

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Event loop](../04-javascript/09-event-loop.md), [Asynchroniczność](../04-javascript/08-asynchronicznosc.md)

JavaScript na stronie działa w jednym wątku — ale platforma pozwala uruchomić **prawdziwe równoległe wątki**: Web Workers. Worker nie ma dostępu do DOM; komunikuje się z main threadem przez wiadomości. To narzędzie na obliczenia, które inaczej zamroziłyby UI.

## Kiedy worker, a kiedy nie

**Async ≠ równoległość:** `await`/Promise nie przenosi obliczeń poza wątek — tylko je przeplata ([event loop](../04-javascript/09-event-loop.md)). Pętla licząca 2 s zablokuje UI niezależnie od async. Reguła:

- **I/O (sieć, timery)** → zwykła asynchroniczność.
- **CPU (parsowanie wielkich JSON/CSV, kompresja, obróbka obrazu/audio, krypto, diffowanie, wyszukiwanie w dużych zbiorach)** → worker.
- Praca < ~50 ms → nie opłaca się (koszt komunikacji); spróbuj najpierw chunkingu z yield ([optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)).

## Dedicated Worker — podstawy

```js
// main.js
const worker = new Worker(new URL('./heavy.worker.js', import.meta.url), { type: 'module' });
worker.postMessage({ cmd: 'parse', payload: bigText });
worker.onmessage = (e) => render(e.data);
worker.onerror = (e) => report(e);
worker.terminate();               // zabij, gdy niepotrzebny

// heavy.worker.js
self.onmessage = (e) => {
  const result = heavyParse(e.data.payload);
  self.postMessage(result);
};
```

- Wzorzec `new URL(..., import.meta.url)` rozumieją bundlery ([Vite/webpack](../14-narzedzia/03-bundlery-i-proces-budowania.md)) — wydzielą chunk workera.
- Worker ma własny global (`self`), event loop i zakres — **zero współdzielonej pamięci** (poza SAB, niżej).
- Dostępne w workerze: `fetch`, `IndexedDB` ([przechowywanie](./04-przechowywanie-danych.md)), timery, WebSocket, OffscreenCanvas, crypto. **Niedostępne: DOM, window, localStorage.**

## Komunikacja: kopiowanie vs transfer

`postMessage` **kopiuje** dane algorytmem structured clone (obiekty, Map, ArrayBuffer; bez funkcji/DOM). Kopiowanie 100 MB też kosztuje — duże bufory **transferuj** (zmiana właściciela, zero kopii):

```js
worker.postMessage({ buffer }, [buffer]);   // transferable: ArrayBuffer, OffscreenCanvas…
// po transferze buffer po stronie nadawcy jest odłączony (nieużywalny)
```

- Ergonomia: surowe postMessage szybko rośnie w spaghetti — biblioteka **Comlink** zamienia workera w „zdalny obiekt" z await.
- `SharedArrayBuffer` + `Atomics` — prawdziwa współdzielona pamięć; wymaga nagłówków cross-origin isolation (COOP/COEP); nisza (WASM, gry, audio).

## Pozostałe typy workerów

- **Shared Worker** — jeden proces współdzielony przez wiele kart tego samego origin (rzadko używany, brak na niektórych mobile).
- **Service Worker** — to NIE narzędzie do obliczeń: programowalne proxy sieciowe (offline, cache, push). Osobny temat: [PWA i service worker](../08-architektura-aplikacji/02-spa-mpa-pwa.md).
- **Worklets** (Audio, Paint) — wyspecjalizowane mini-wątki dla API mediów/CSS.

## Wzorce produkcyjne

- **Pula workerów** — `navigator.hardwareConcurrency` podpowiada liczbę rdzeni; kolejka zadań → wolny worker (analogia do puli połączeń).
- **Request/response z id** — do wiadomości dokładaj `id`, odpowiedź mapuj na Promise (Comlink robi to za Ciebie).
- Workery a stan: przenoś **czyste obliczenia** ([funkcje czyste](../04-javascript/02-funkcje.md)); synchronizacja stanu UI w workerze to walka pod prąd.
- Debugowanie: DevTools → Sources → wątki workerów widoczne osobno ([DevTools](../14-narzedzia/05-devtools.md)).

## Pułapki i częste błędy

- Wrzucenie do workera pracy I/O-bound — zero zysku, dodatkowa złożoność.
- Przesyłanie ogromnych obiektów bez transferu — koszt klonowania zjada zysk z równoległości.
- Oczekiwanie dostępu do DOM/localStorage w workerze.
- Tworzenie workera per zadanie zamiast puli — koszt startu (~ms) przy częstych małych zadaniach.
- Brak terminate/cleanup — wątki-zombie.
- Zapominanie, że worker to osobny plik/chunk — ścieżki i build wymagają wsparcia bundlera.

## Pytania rekrutacyjne

1. **Async/await vs Web Worker — kiedy które?** — przeplatanie I/O vs prawdziwa równoległość CPU.
2. **Jak worker komunikuje się z main threadem i co można przesłać?** — postMessage + structured clone; transferables dla zero-copy.
3. **Czego worker nie może i dlaczego?** — DOM/window: DOM nie jest thread-safe.
4. **Różnica Web Worker vs Service Worker?** — obliczenia vs proxy sieciowe/cykl życia niezależny od karty.
5. **Kiedy NIE użyjesz workera mimo ciężkich obliczeń?** — małe zadania (narzut), potrzeba DOM; wtedy chunking+yield.

## Dalsza lektura

- [MDN: Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)
- [Comlink](https://github.com/GoogleChromeLabs/comlink)
- [web.dev: Off the main thread](https://web.dev/articles/off-main-thread)
