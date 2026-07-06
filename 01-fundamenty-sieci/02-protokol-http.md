# Protokół HTTP

> **Poziom:** 🟢 podstawowy (podstawy) → 🟡 średni (wersje protokołu, caching)
> **Wymagana wiedza:** [Jak działa internet](./01-jak-dziala-internet.md)

HTTP (Hypertext Transfer Protocol) to protokół warstwy aplikacji, w którym klient (przeglądarka) wysyła **request**, a serwer zwraca **response**. Jest **bezstanowy** — każdy request jest niezależny, a „pamięć" (sesje, logowanie) buduje się na nim za pomocą cookies i tokenów. To najważniejszy protokół w pracy frontend developera.

## Anatomia requestu i response'u

```http
GET /api/users?page=2 HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer eyJhbGci...
Cookie: session_id=abc123
```

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Cache-Control: max-age=3600
Set-Cookie: session_id=abc123; HttpOnly; Secure

{"users": [...]}
```

Request = metoda + ścieżka + nagłówki + (opcjonalnie) body. Response = **status** + nagłówki + (opcjonalnie) body.

## Metody HTTP

| Metoda | Zastosowanie | Idempotentna? | Bezpieczna? | Ma body? |
|---|---|---|---|---|
| `GET` | pobranie zasobu | tak | tak | nie |
| `POST` | utworzenie zasobu / akcja | **nie** | nie | tak |
| `PUT` | pełna podmiana zasobu | tak | nie | tak |
| `PATCH` | częściowa modyfikacja | nie (z definicji) | nie | tak |
| `DELETE` | usunięcie | tak | nie | zwykle nie |
| `HEAD` | jak GET, ale bez body | tak | tak | nie |
| `OPTIONS` | możliwości serwera (m.in. CORS preflight) | tak | tak | nie |

- **Bezpieczna (safe)** = nie zmienia stanu serwera. **Idempotentna** = wielokrotne wykonanie daje ten sam efekt co jednokrotne (kluczowe przy retry!).
- `OPTIONS` zobaczysz głównie jako **preflight** przy CORS — szczegóły w [mechanizmach obronnych](../11-bezpieczenstwo/02-mechanizmy-obronne.md).
- Jak metody mapują się na projektowanie API: [REST](../12-komunikacja-z-api/01-rest.md).

## Kody statusów

- **1xx** — informacyjne (np. `101 Switching Protocols` przy WebSocket).
- **2xx** — sukces: `200 OK`, `201 Created`, `204 No Content` (sukces bez body).
- **3xx** — przekierowania: `301` (stałe), `302`/`307` (tymczasowe; `307` zachowuje metodę), `304 Not Modified` (odpowiedź na request warunkowy — patrz caching niżej).
- **4xx** — błąd klienta: `400 Bad Request`, `401 Unauthorized` (brak/nieważne uwierzytelnienie), `403 Forbidden` (uwierzytelniony, ale bez uprawnień), `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity` (błędy walidacji), `429 Too Many Requests`.
- **5xx** — błąd serwera: `500`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout`.

Praktyczna różnica dla frontendu: **`fetch` nie rzuca błędu przy 4xx/5xx** — trzeba sprawdzić `response.ok` ([fetch w praktyce](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).

## Najważniejsze nagłówki

- **Negocjacja treści:** `Accept`, `Accept-Language`, `Accept-Encoding` (gzip/brotli) ↔ `Content-Type`, `Content-Encoding`.
- **Stan:** `Cookie` ↔ `Set-Cookie` (flagi `HttpOnly`, `Secure`, `SameSite` — omówione przy [uwierzytelnianiu](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)).
- **Cache:** `Cache-Control`, `ETag`, `Last-Modified` (niżej).
- **Bezpieczeństwo:** `Content-Security-Policy`, `Strict-Transport-Security` i inne — dom: [mechanizmy obronne](../11-bezpieczenstwo/02-mechanizmy-obronne.md).
- **CORS:** `Origin` ↔ `Access-Control-Allow-Origin` — tamże.

## HTTP caching (to jest „dom" tego tematu — inne pliki tu linkują)

Cache HTTP decyduje, czy przeglądarka w ogóle wyśle request. Steruje nim nagłówek **`Cache-Control`**:

```http
Cache-Control: max-age=31536000, immutable   # statyki z hashem w nazwie
Cache-Control: no-cache                       # cache'uj, ale zawsze rewaliduj
Cache-Control: no-store                       # nie cache'uj wcale (dane wrażliwe)
Cache-Control: private, max-age=60            # tylko cache przeglądarki, nie CDN
Cache-Control: s-maxage=3600                  # TTL dla cache współdzielonych (CDN)
```

**Dwa tryby działania:**

1. **Fresh (świeży)** — dopóki nie minie `max-age`, przeglądarka używa kopii **bez żadnego requestu** (w DevTools: „from disk cache" / „from memory cache").
2. **Stale (przeterminowany) → rewalidacja** — przeglądarka wysyła **request warunkowy**:
   - `If-None-Match: "<etag>"` (porównanie z `ETag` — odcisk treści) lub
   - `If-Modified-Since: <data>` (porównanie z `Last-Modified`).
   Jeśli zasób się nie zmienił, serwer odpowiada **`304 Not Modified`** bez body — oszczędza transfer, ale nie RTT.

**Złota strategia dla frontendu:**
- HTML: `no-cache` (zawsze rewaliduj) — bo to on wskazuje wersje pozostałych plików.
- JS/CSS/obrazy **z hashem w nazwie** (`app.3f2a1b.js`): `max-age=31536000, immutable` — cache na rok, zmiana treści = nowa nazwa = nowy URL (cache busting).

Jak to się składa z CDN i Service Workerem w całą architekturę cache: [strategie cache'owania](../09-wydajnosc/04-strategie-cachowania.md).

## HTTP/1.1 vs HTTP/2 vs HTTP/3

| | HTTP/1.1 (1997) | HTTP/2 (2015) | HTTP/3 (2022) |
|---|---|---|---|
| Transport | TCP | TCP | **QUIC (UDP)** |
| Równoległość | 1 request na połączenie (przeglądarki otwierają ~6 połączeń/host) | **multipleksacja** — wiele strumieni w 1 połączeniu | multipleksacja bez blokad |
| Główny problem | head-of-line blocking na poziomie HTTP | HoL blocking na poziomie **TCP** (zgubiony pakiet blokuje wszystkie strumienie) | rozwiązany — strumienie niezależne |
| Nagłówki | tekstowe, powtarzane | binarne, kompresja HPACK | kompresja QPACK |
| Dodatki | keep-alive, pipelining (martwy) | priorytetyzacja strumieni | 0-RTT, migracja połączeń (zmiana sieci bez zrywania) |

**Praktyczne konsekwencje dla frontendu:**
- Ery HTTP/1.1 nawyki — sprite'y CSS, konkatenacja wszystkiego w 1 plik, domain sharding — z HTTP/2+ są **zbędne lub szkodliwe**.
- Multipleksacja nie znosi jednak kosztu *ilości bajtów* ani *łańcuchów zależności* (plik A importuje B, B importuje C) — zob. [optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md).

## Pułapki i częste błędy

- Używanie `GET` do operacji zmieniających stan — GET może być prefetchowany/cache'owany przez pośredników; skutki bywają zabawne i katastrofalne.
- Retry requestów nieidempotentnych (`POST`) bez klucza idempotencji — ryzyko podwójnych zamówień/płatności.
- `no-cache` ≠ „nie cache'uj" (to `no-store`); `no-cache` znaczy „cache'uj, ale rewaliduj przed użyciem".
- Deployment nowej wersji bez cache bustingu — użytkownicy z HTML nowej wersji proszą o stare chunki JS (lub odwrotnie) i dostają błędy.
- Mylenie `401` z `403`: `401` = „nie wiem, kim jesteś", `403` = „wiem, ale nie wolno ci".

## Pytania rekrutacyjne

1. **Co znaczy, że HTTP jest bezstanowy i jak buduje się na nim sesje?** — każdy request niezależny; stan przez cookies (session id) lub tokeny w nagłówkach.
2. **Różnica między `PUT` a `PATCH`?** — PUT podmienia cały zasób (idempotentny), PATCH modyfikuje częściowo.
3. **Jak działa `ETag` i status `304`?** — odcisk treści; request warunkowy `If-None-Match`; 304 = „użyj swojej kopii", brak body.
4. **Co daje HTTP/2 i jaki problem został dopiero w HTTP/3 rozwiązany?** — multipleksacja, kompresja nagłówków; TCP head-of-line blocking rozwiązany przez QUIC.
5. **Jak skonfigurować cache dla SPA po buildzie?** — HTML `no-cache`, assety z hashem `max-age=31536000, immutable`.
6. **Kiedy przeglądarka wysyła request OPTIONS?** — CORS preflight dla requestów „niezwykłych" (custom headers, metody inne niż proste) — [szczegóły](../11-bezpieczenstwo/02-mechanizmy-obronne.md).

## Dalsza lektura

- [MDN: HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) — kompletna dokumentacja nagłówków i statusów
- [MDN: HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching)
- [web.dev: HTTP cache — praktyczny przewodnik](https://web.dev/articles/http-cache)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) (gdy chcesz źródła prawdy)
