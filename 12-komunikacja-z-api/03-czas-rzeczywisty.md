# Komunikacja w czasie rzeczywistym

> **Poziom:** 🟡 średni → 🔴 zaawansowany
> **Wymagana wiedza:** [Protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md), [Fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md), [Jak działa internet](../01-fundamenty-sieci/01-jak-dziala-internet.md)

Klasyczny HTTP jest jednokierunkowy: klient pyta, serwer odpowiada. Gdy to **serwer** ma coś przekazać (czat, powiadomienia, notowania, współpraca na żywo, streaming odpowiedzi AI), potrzebujesz innego mechanizmu. Są trzy główne — różnią się kierunkiem, kosztem i prostotą.

## Techniki — porównanie

| | Polling | Long polling | SSE | WebSocket |
|---|---|---|---|---|
| Kierunek | klient pyta | klient pyta (serwer trzyma) | serwer → klient | **dwukierunkowy** |
| Transport | HTTP (wiele req) | HTTP | HTTP (jeden, strumień) | osobny protokół (ws://) |
| Reconnect | trywialny | trywialny | **automatyczny (wbudowany)** | ręczny |
| Złożoność | najniższa | niska | niska | wyższa (stan połączenia) |
| Typ danych | dowolny | dowolny | tekst (UTF-8) | tekst + binarny |
| Skalowanie | kosztowne (dużo req) | trzyma połączenia | proste (zwykły HTTP) | wymaga sticky/infra |

## Polling i long polling

**Polling** — `setInterval` + fetch co X sekund. Prosty, uniwersalny, ale kompromis: krótki interwał = obciążenie i koszt, długi = opóźnienie. Dobre dla danych zmieniających się rzadko/przewidywalnie (status zamówienia co 30 s). Pamiętaj o pauzie w tle (`visibilitychange` — [przydatne API](../05-dom-i-web-api/07-przydatne-api-przegladarki.md)) i o [AbortController](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md).

**Long polling** — klient wysyła request, serwer **trzyma** go otwartym aż pojawi się dane (lub timeout), potem klient natychmiast pyta ponownie. Zbliżone do „real-time" bez WebSocketów; historyczny fundament, dziś raczej fallback.

## SSE (Server-Sent Events)

Jednokierunkowy strumień serwer→klient po zwykłym HTTP, natywne API `EventSource`:

```js
const es = new EventSource('/api/stream');
es.onmessage = (e) => update(JSON.parse(e.data));
es.addEventListener('price', (e) => …);   // nazwane typy zdarzeń
es.onerror = () => { /* EventSource sam się reconnectuje */ };
// serwer wysyła: "event: price\ndata: {...}\n\n"; "id:" wspiera Last-Event-ID przy wznowieniu
```

- ✚ **automatyczny reconnect** i wznawianie (`Last-Event-ID`), działa przez zwykłą infrastrukturę HTTP/[proxy](../01-fundamenty-sieci/04-domeny-hosting-cdn.md), prosty.
- ✖ tylko serwer→klient, tylko tekst, limit połączeń na domenę w HTTP/1.1 (mniej istotny w HTTP/2).
- Idealny do: powiadomień, feedów, notowań, **streamingu odpowiedzi LLM token po tokenie**, pasków postępu. Gdy nie potrzebujesz kanału zwrotnego — SSE bije WebSocket prostotą.

## WebSocket

Pełnodupleksowe, trwałe połączenie po osobnym protokole (upgrade z HTTP — [status 101](../01-fundamenty-sieci/02-protokol-http.md)):

```js
const ws = new WebSocket('wss://example.com/socket');   // wss = szyfrowane (jak https)
ws.onopen = () => ws.send(JSON.stringify({ type: 'join', room: 42 }));
ws.onmessage = (e) => handle(JSON.parse(e.data));
ws.onclose = () => scheduleReconnect();                 // reconnect ROBISZ SAM
```

- ✚ dwukierunkowość, niska latencja, dane binarne — czaty, gry, wspólna edycja, tablice live.
- ✖ musisz sam ogarnąć: **reconnect z backoffem**, heartbeat/ping-pong (wykrywanie zerwania), kolejkowanie wiadomości przy rozłączeniu, resync stanu po powrocie, uwierzytelnianie (token w pierwszej wiadomości/query — nie w nagłówkach), skalowanie (sticky sessions/pub-sub na backendzie).
- W praktyce często biblioteka: **Socket.IO** (reconnect+fallbacki+pokoje), a do współpracy na żywo — CRDT (**Yjs**) nad WebSocketem.

## Jak wybrać (drzewo decyzyjne)

1. Serwer nie musi pushować, wystarczy odświeżanie → **polling** (najprościej).
2. Tylko serwer→klient (powiadomienia, feed, streaming AI) → **SSE**.
3. Prawdziwa dwukierunkowość, niska latencja (czat, gra, współpraca) → **WebSocket** (+ biblioteka).
4. Nie zaczynaj od WebSocketów „bo real-time" — to najdroższa operacyjnie opcja; wiele „real-time" potrzeb obsługuje SSE lub nawet polling.

WebTransport (nad HTTP/3) to następca na horyzoncie — kojarz nazwę.

## Wzorce niezależne od techniki

- **Reconnect z exponential backoff + jitter** (nie zalewaj serwera po awarii), cap na maksymalny interwał.
- **Resync po ponownym połączeniu** — pobierz aktualny stan (real-time zakłada, że mogłeś przegapić wiadomości).
- **Integracja ze stanem/cache** — wiadomości aktualizują ten sam cache co REST/GraphQL ([wzorce pracy z danymi](./04-wzorce-pracy-z-danymi.md)); jedno źródło prawdy, nie dwa równoległe.
- **Sprzątanie** — zamknij `es.close()`/`ws.close()` przy unmount ([memory leaki](../04-javascript/12-zaawansowane.md)); pauzuj w tle.
- **Backpressure** — throttle aktualizacji UI przy zalewie wiadomości (notowania 100/s → render raz na klatkę; [optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)).

## Pułapki i częste błędy

- WebSocket bez reconnect/heartbeat — połączenie „umiera" cicho (NAT/proxy zrywają idle), UI zamarza.
- Brak resync po reconnect — użytkownik widzi stan sprzed rozłączenia.
- Real-time jako drugie, rozjechane źródło prawdy obok REST.
- SSE/WS niesprzątane przy nawigacji SPA — mnożące się połączenia.
- Auth WebSocketu jak REST (nagłówki) — WS nie ma customowych nagłówków z przeglądarki; token przez protokół/query/pierwszą wiadomość.
- Zalew renderów przy dużej częstotliwości wiadomości (brak batchingu/throttlingu).

## Pytania rekrutacyjne

1. **Polling vs SSE vs WebSocket — kiedy które?** — kierunek, koszt, złożoność; drzewo decyzyjne.
2. **Co SSE daje za darmo względem WebSocket?** — auto-reconnect, wznawianie, zwykły HTTP; ograniczenia (jednokierunkowy, tekst).
3. **Co musisz sam zaimplementować dla produkcyjnego WebSocketu?** — reconnect+backoff, heartbeat, resync, auth, skalowanie.
4. **Jak streamowałbyś odpowiedź modelu AI token po tokenie?** — SSE (lub strumień [fetch/ReadableStream](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).
5. **Jak integrujesz dane real-time ze stanem aplikacji?** — aktualizacja wspólnego cache; jedno źródło prawdy; throttling UI.

## Dalsza lektura

- [MDN: Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) / [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Ably: SSE vs WebSockets vs long polling](https://ably.com/blog/websockets-vs-sse)
- [Yjs (CRDT do współpracy na żywo)](https://docs.yjs.dev/)
