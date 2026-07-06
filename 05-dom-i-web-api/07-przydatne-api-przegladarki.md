# Przydatne API przeglądarki

> **Poziom:** 🟡 średni (przegląd)
> **Wymagana wiedza:** [Zdarzenia](./02-zdarzenia.md), [Asynchroniczność](../04-javascript/08-asynchronicznosc.md)

Przegląd mniejszych API, które regularnie pojawiają się w pracy — od animacji po schowek. Nie musisz znać każdego szczegółu; musisz **wiedzieć, że istnieją**, żeby nie pisać ich na piechotę ani nie instalować biblioteki do rzeczy wbudowanej.

## requestAnimationFrame

Callback **przed następnym paintem**, zsynchronizowany z odświeżaniem ekranu:

```js
function tick(timestamp) {
  update(timestamp);
  rafId = requestAnimationFrame(tick);
}
let rafId = requestAnimationFrame(tick);
cancelAnimationFrame(rafId);
```

- Do animacji sterowanych JS (canvas, liczniki, drag) — zamiast `setInterval(…, 16)` (dryfuje, nie synchronizuje się z klatkami; [event loop](../04-javascript/09-event-loop.md)).
- W nieaktywnej karcie rAF się zatrzymuje — darmowe „pauzuj w tle".
- Deklaratywne animacje CSS zwykle lepsze ([animacje](../03-css/07-animacje-i-transformacje.md)); rAF gdy potrzebna logika per klatka.
- `requestIdleCallback` — praca w czasie bezczynności (niskopriorytetowe: prefetch, telemetria); nowszy `scheduler.postTask` z priorytetami.

## History API — fundament routingu SPA

```js
history.pushState({ id: 42 }, '', '/products/42');   // zmiana URL BEZ przeładowania
history.replaceState(state, '', url);                  // podmiana bieżącego wpisu
window.addEventListener('popstate', e => render(e.state));  // wstecz/dalej
```

Na tym stoją routery SPA — pełny obraz (guards, lazy routes, scroll restoration): [routing po stronie klienta](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md). Nowa **Navigation API** (przechwytywanie nawigacji jednym interfejsem) stopniowo to upraszcza.

## Clipboard API

```js
await navigator.clipboard.writeText(text);            // wymaga HTTPS + gestu użytkownika
const text = await navigator.clipboard.readText();    // odczyt: za zgodą (prompt)
```

Zapis zwykle działa po kliknięciu; odczyt jest mocniej strzeżony. Obsłuż odmowę (try/catch). Stary `document.execCommand('copy')` — deprecated.

## Widoczność strony i cykl życia karty

```js
document.addEventListener('visibilitychange', () => {
  if (document.hidden) pauseVideo(); else resume();
});
```

- `visibilitychange` — karta w tle: pauzuj media/polling, wysyłaj telemetrię (`navigator.sendBeacon(url, data)` — request, który przeżyje zamknięcie strony; standard dla analityki wyjścia).
- `pagehide`/`pageshow` zamiast `unload` (unload psuje bfcache — natychmiastowy „wstecz"; [optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md)).
- `online`/`offline` + `navigator.onLine` — sygnał (niedoskonały) stanu sieci.

## Powiadomienia i uprawnienia

```js
const perm = await Notification.requestPermission();   // pytaj PO geście i z kontekstem!
if (perm === 'granted') new Notification('Gotowe', { body: '…' });
```

- Push w tle (gdy strona zamknięta) = Notification + Push API + Service Worker ([PWA](../08-architektura-aplikacji/02-spa-mpa-pwa.md)).
- Wzorzec UX: najpierw własny, „miękki" prompt w UI; natywny dialog dopiero po zgodzie — odrzucony natywny prompt często blokuje na zawsze.
- Ten sam model uprawnień: geolokalizacja (`navigator.geolocation.getCurrentPosition`), kamera/mikrofon (`getUserMedia`), `navigator.permissions.query({name})` do sprawdzania stanu.

## Drobiazgi, które warto kojarzyć

- **`<dialog>` i Popover API** — modale/popovery natywnie ([nowoczesny CSS](../03-css/08-nowoczesny-css.md)).
- **Web Share:** `navigator.share({ title, url })` — natywny arkusz udostępniania (mobile).
- **Fullscreen:** `el.requestFullscreen()`; **Screen Wake Lock:** `navigator.wakeLock.request()` (ekran nie gaśnie — przepisy kulinarne, prezentacje).
- **`matchMedia`** — media queries w JS: `matchMedia('(max-width: 48rem)').addEventListener('change', …)` (nie duplikuj breakpointów „na sztywno").
- **URL/URLSearchParams** — parsowanie i budowa URL-i ([anatomia URL](../01-fundamenty-sieci/04-domeny-hosting-cdn.md)).
- **Intl** — formatowanie dat/liczb/walut/plural (`Intl.NumberFormat('pl-PL', { style: 'currency', currency: 'PLN' })`) — nigdy ręcznie.
- **crypto:** `crypto.randomUUID()`, `crypto.getRandomValues()` — id i losowość (nie `Math.random()` do niczego bezpieczeństwowego).
- **BroadcastChannel** — komunikacja między kartami ([przechowywanie](./04-przechowywanie-danych.md)).
- **Drag and Drop API / File API** — upload przeciągnij-upuść (`dataTransfer.files`, `FileReader`/`file.arrayBuffer()`).

## Pułapki i częste błędy

- Wywoływanie API wymagających **gestu użytkownika** (clipboard, fullscreen, autoplay z dźwiękiem) poza handlerem interakcji — cicha odmowa.
- Zapominanie o **HTTPS jako warunku** większości nowoczesnych API ([TLS](../01-fundamenty-sieci/01-jak-dziala-internet.md)).
- Pytanie o powiadomienia/geolokalizację od razu po wejściu — masowe odmowy.
- Feature detection przez sniffing User-Agenta zamiast `if ('share' in navigator)`.
- Polling zamiast eventów (`visibilitychange`, `online`) i obserwerów ([obserwery](./05-observery.md)).

## Pytania rekrutacyjne

1. **rAF vs setInterval do animacji?** — synchronizacja z klatkami, pauza w tle, brak dryfu.
2. **Jak działa routing SPA bez przeładowań?** — pushState/popstate; co trzeba doimplementować ([routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)).
3. **Jak niezawodnie wysłać dane analityczne przy zamykaniu strony?** — sendBeacon + visibilitychange/pagehide (nie unload).
4. **Czemu `navigator.clipboard.readText()` może nie zadziałać?** — uprawnienia, brak gestu, HTTP.
5. **Jak wykryć wsparcie funkcji przeglądarki?** — feature detection (`in`, `@supports`), nie User-Agent.

## Dalsza lektura

- [MDN: Web APIs — indeks](https://developer.mozilla.org/en-US/docs/Web/API)
- [web.dev: Capabilities (Project Fugu)](https://developer.chrome.com/docs/capabilities)
- [What Web Can Do Today](https://whatwebcando.today/) — przegląd możliwości platformy
