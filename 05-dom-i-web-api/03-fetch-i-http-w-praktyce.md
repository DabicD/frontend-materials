# Fetch — HTTP w praktyce

> **Poziom:** 🟡 średni (dom tematu fetch — inne pliki tu linkują)
> **Wymagana wiedza:** [Protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md), [Asynchroniczność](../04-javascript/08-asynchronicznosc.md)

`fetch` to standardowe API do requestów HTTP. Wygląda banalnie, ale ma kilka ostrych krawędzi (błędy, które nie rzucają; strumieniowe body; anulowanie), których nieznajomość produkuje subtelne bugi. Wzorce wyższego poziomu (retry, cache, race conditions): [wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md).

## Podstawy i pułapka nr 1

```js
const res = await fetch('/api/users');

// fetch odrzuca Promise TYLKO przy błędzie SIECI (offline, DNS, CORS).
// Status 404/500 to POPRAWNA odpowiedź — trzeba sprawdzić samemu:
if (!res.ok) {                       // ok ⇔ status 200–299
  throw new HttpError(res);          // własna klasa — patrz obsługa błędów
}
const data = await res.json();       // body czyta się ASYNCHRONICZNIE (strumień)
```

Obiekt `Response`: `status`, `ok`, `headers` (iterowalne: `res.headers.get('content-type')`), `url` (po przekierowaniach), body-readery: `json()`, `text()`, `blob()`, `formData()`, `arrayBuffer()`.

**Body można przeczytać tylko raz** (to strumień) — drugie `res.json()` rzuci; potrzebujesz dwa razy → `res.clone()`.

## Konfiguracja requestu

```js
const res = await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Ala' }),   // body NIE przyjmuje obiektu wprost
  credentials: 'same-origin',              // cookies: 'omit' | 'same-origin' | 'include' (cross-origin)
  mode: 'cors',                            // 'cors' | 'same-origin' | 'no-cors' (rzadko!)
  cache: 'no-store',                       // interakcja z HTTP cache
  signal: abortController.signal,          // anulowanie (niżej)
});
```

- **JSON:** ręcznie `JSON.stringify` + nagłówek `Content-Type: application/json`.
- **Cookies cross-origin** wymagają `credentials: 'include'` **i** zgody serwera (CORS z `Access-Control-Allow-Credentials`) — [mechanizmy obronne](../11-bezpieczenstwo/02-mechanizmy-obronne.md).
- Błąd CORS objawia się jako **TypeError „Failed to fetch"** — szczegóły tylko w konsoli (celowo ukryte przed JS).

## FormData i upload plików

```js
const fd = new FormData(formElement);          // z formularza (pola z name!)
fd.append('avatar', fileInput.files[0]);
await fetch('/api/profile', { method: 'POST', body: fd });
// NIE ustawiaj Content-Type ręcznie — przeglądarka doda multipart/form-data z boundary
```

`FormData` jest iterowalne (`[...fd.entries()]`); to też najprostszy sposób odczytu formularza do obiektu: `Object.fromEntries(new FormData(form))`.

## Anulowanie: AbortController

```js
const ac = new AbortController();
fetch(url, { signal: ac.signal });
ac.abort();                          // → fetch odrzuca DOMException 'AbortError'

// timeout jednym wyrazem:
fetch(url, { signal: AbortSignal.timeout(5000) });
// łączenie powodów: AbortSignal.any([ac.signal, AbortSignal.timeout(5000)])
```

Anuluj, gdy: użytkownik zmienił widok, wpisał kolejną literę w wyszukiwarkę (poprzedni request już nieaktualny), komponent umiera. Obsługa: odróżnij `AbortError` od prawdziwych błędów (nie pokazuj „błąd sieci" po własnym anulowaniu!). To podstawowe narzędzie przeciw race conditions ([wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)).

## Strumienie i postęp

`res.body` to `ReadableStream` — można przetwarzać w trakcie pobierania:

```js
const reader = res.body.getReader();
let received = 0;
for (;;) {
  const { done, value } = await reader.read();
  if (done) break;
  received += value.length;                    // postęp pobierania
  onProgress(received / Number(res.headers.get('content-length')));
}
```

Zastosowania: progres dużych pobrań, NDJSON/odpowiedzi generowane strumieniowo (czaty AI), wczesne przetwarzanie. **Postępu uploadu** fetch (jeszcze) nie wystawia — to ostatnia nisza starego `XMLHttpRequest` (`xhr.upload.onprogress`); poza nią XHR to legacy.

## Warstwa API w aplikacji (praktyka)

Nie rozsiewaj gołych `fetch` po komponentach — jedna warstwa (fasada — [wzorce](../04-javascript/13-wzorce-projektowe-js.md)):

```js
async function api(path, { body, ...options } = {}) {
  const res = await fetch(`${BASE_URL}${path}`, {
    headers: { 'Content-Type': 'application/json', ...authHeader() },
    body: body && JSON.stringify(body),
    ...options,
  });
  if (res.status === 401) return handleSessionExpired();
  if (!res.ok) throw await HttpError.fromResponse(res);
  return res.status === 204 ? null : res.json();
}
```

Centralizuje: bazowy URL, auth, obsługę błędów ([strategia](../04-javascript/11-obsluga-bledow.md)), parsowanie, retry.

## Pułapki i częste błędy

- Brak sprawdzenia `res.ok` — `res.json()` z odpowiedzi błędu „działa", a potem `undefined is not a function` daleko od przyczyny.
- Podwójne czytanie body bez `clone()`.
- Ręczny `Content-Type` przy `FormData` (psuje boundary) albo jego brak przy JSON.
- Zdziwienie, że cookies nie idą cross-origin bez `credentials: 'include'` + zgody CORS.
- Nieanulowane requesty poprzednich widoków nadpisują nowsze dane (race condition).
- `mode: 'no-cors'` jako „obejście CORS" — dostajesz opaque response bez dostępu do czegokolwiek; to nie jest rozwiązanie.

## Pytania rekrutacyjne

1. **Kiedy fetch odrzuca Promise, a kiedy nie?** — tylko błędy sieci/CORS/abort; statusy HTTP wymagają `res.ok`.
2. **Jak anulować request i po co?** — AbortController/AbortSignal.timeout; sprzątanie, wyszukiwarki, race conditions.
3. **Dlaczego body odpowiedzi czyta się async i tylko raz?** — strumień; clone() do wielokrotnego odczytu.
4. **Jak wysłać plik z paskiem postępu?** — FormData; upload progress → XHR (nisza) lub podejścia strumieniowe.
5. **Jak zaprojektujesz warstwę HTTP w aplikacji?** — fasada: auth, błędy→domenowe, retry, typowanie odpowiedzi ([TypeScript](../06-typescript/05-typescript-w-praktyce.md)).

## Dalsza lektura

- [MDN: Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [javascript.info: Network requests](https://javascript.info/network)
- [web.dev: Streams — the definitive guide](https://web.dev/articles/streams)
