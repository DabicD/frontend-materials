# Przechowywanie danych w przeglądarce

> **Poziom:** 🟢 podstawowy → 🟡 średni (IndexedDB, decyzje)
> **Wymagana wiedza:** [Protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md) (cookies)

Przeglądarka daje kilka mechanizmów przechowywania — różnią się pojemnością, czasem życia, dostępnością z JS i tym, czy podróżują z requestami. Wybór ma konsekwencje bezpieczeństwa (gdzie trzymać tokeny? — [uwierzytelnianie](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)) i architektoniczne (offline — [PWA](../08-architektura-aplikacji/02-spa-mpa-pwa.md)).

## Porównanie — tabela decyzyjna

| | Cookies | localStorage | sessionStorage | IndexedDB | Cache API |
|---|---|---|---|---|---|
| Pojemność | ~4 KB | ~5–10 MB | ~5 MB | setki MB+ (quota) | jw. |
| Wysyłane do serwera | **tak, każdy request** | nie | nie | nie | nie |
| Dostęp z JS | tak (bez `HttpOnly`) | tak | tak | tak (async) | tak (async) |
| Czas życia | wg `Expires`/sesja | trwałe | karta/sesja | trwałe | trwałe |
| API | string, uciążliwe | sync, proste | sync | async, transakcyjne | async |
| Typ danych | string | string | string | obiekty/pliki (structured clone) | Response'y |
| Dostępne w workerach | — | nie | nie | tak | tak (Service Worker) |

Wszystko jest **per origin** ([origin](../01-fundamenty-sieci/04-domeny-hosting-cdn.md)) — subdomena to inny magazyn (cookies mogą być współdzielone przez `Domain=`).

## Cookies

Jedyny mechanizm, który **automatycznie podróżuje z requestami** — dlatego służy sesjom i uwierzytelnianiu.

```js
document.cookie = 'theme=dark; Max-Age=31536000; Path=/; Secure; SameSite=Lax';
```

- Atrybuty: `Max-Age`/`Expires` (bez nich — cookie sesyjne), `Path`, `Domain` (z kropką = subdomeny), **`Secure`** (tylko HTTPS), **`HttpOnly`** (niedostępne z JS — ustawiane tylko przez serwer, ochrona przed kradzieżą przez XSS), **`SameSite`** (`Lax`/`Strict`/`None` — obrona CSRF; szczegóły: [uwierzytelnianie](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)).
- JS-owe API `document.cookie` jest archaiczne (string parsing); nowsze `cookieStore` (async) — wsparcie rosnące.
- Koszt: cookies doklejają się do **każdego** requestu na origin — kilobajty nagłówków na request.

## Web Storage: localStorage i sessionStorage

```js
localStorage.setItem('settings', JSON.stringify(settings));
const settings = JSON.parse(localStorage.getItem('settings') ?? '{}');
localStorage.removeItem('settings'); localStorage.clear();
```

- **Tylko stringi** — obiekty przez `JSON.stringify/parse` (gubi Date/Map itd. — [ES6+](../04-javascript/07-skladnia-es6-plus.md)).
- **Synchroniczne** — duże odczyty/zapisy blokują main thread; nie nadaje się na „bazę danych".
- `sessionStorage` żyje per **karta** (duplikacja karty kopiuje stan!); przetrwa reload, nie przetrwa zamknięcia.
- Zdarzenie `storage` — strzela w **innych kartach** tego samego origin przy zmianie: prosty kanał synchronizacji między kartami (nowocześniej: `BroadcastChannel`).
- Prywatne okna: storage działa, znika po zamknięciu; iOS bywa restrykcyjny — zawsze obsłuż wyjątki (quota/`SecurityError`).

**Typowe zastosowania:** preferencje UI (motyw, język), drafty formularzy, flagi onboardingu. **Nie:** tokeny (XSS czyta localStorage — [dyskusja](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)), duże dane, nic krytycznego (użytkownik może wyczyścić).

## IndexedDB

Transakcyjna, asynchroniczna baza obiektów: object stores + indeksy + kursory; przechowuje obiekty, pliki (Blob), duże wolumeny.

- Natywne API jest niskopoziomowe i callbackowe — w praktyce używa się wrapperów: **`idb`** (Promise), **Dexie**, lub gotowych warstw (offline cache bibliotek data-fetchingowych).
- Zastosowania: offline-first (kolejka mutacji do wysłania po odzyskaniu sieci), cache dużych odpowiedzi API, dane binarne, aplikacje typu edytor.
- Dostępna w [Web Workers](./06-web-workers.md) i Service Workerze — przetwarzanie danych poza main threadem.

```js
import { openDB } from 'idb';
const db = await openDB('shop', 1, {
  upgrade(db) { db.createObjectStore('products', { keyPath: 'id' }); },
});
await db.put('products', { id: 1, name: 'Buty' });
const p = await db.get('products', 1);
```

## Cache API i quota

- **Cache API** (`caches.open(...)`) — magazyn par Request→Response; serce strategii offline Service Workera ([strategie cache'owania](../09-wydajnosc/04-strategie-cachowania.md), [PWA](../08-architektura-aplikacji/02-spa-mpa-pwa.md)).
- **Quota:** `navigator.storage.estimate()` — ile wolno; `navigator.storage.persist()` — prośba o trwałość (bez niej przeglądarka może czyścić dane pod presją miejsca — „best effort").

## Pułapki i częste błędy

- Tokeny/sekrety w localStorage — czytelne dla każdego XSS; preferuj cookies `HttpOnly` (kompromisy: [uwierzytelnianie](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)).
- Brak try/catch wokół `JSON.parse` z localStorage (dane spoza kontroli — mogły zostać ręcznie zmienione/uszkodzone) i wokół `setItem` (QuotaExceededError).
- Traktowanie localStorage jako niezawodnej bazy — to cache preferencji, może zniknąć.
- Przechowywanie stanu aplikacji w cookies — pęcznieją requesty.
- Zapominanie o wersjonowaniu struktur (`upgrade` w IndexedDB, klucz `settings_v2` w LS) — stare dane po deploymencie psują apkę.

## Pytania rekrutacyjne

1. **Cookies vs localStorage — fundamentalne różnice?** — podróż z requestami, rozmiar, HttpOnly, czas życia.
2. **Gdzie przechowasz token sesji i dlaczego?** — dyskusja cookie HttpOnly vs storage; XSS vs CSRF ([pełna analiza](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)).
3. **Kiedy sięgniesz po IndexedDB?** — duże/strukturalne dane, offline, dostęp z workerów, transakcje.
4. **Jak zsynchronizować stan między kartami?** — event `storage` / BroadcastChannel.
5. **Czym różni się sessionStorage od localStorage?** — zakres karty vs trwały; duplikacja kart.

## Dalsza lektura

- [MDN: Client-side storage](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Client-side_APIs/Client-side_storage)
- [web.dev: Storage for the web](https://web.dev/articles/storage-for-the-web)
- [idb — wrapper IndexedDB](https://github.com/jakearchibald/idb)
