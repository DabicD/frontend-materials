# REST API

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md), [Fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)

REST to najczęstszy styl API, jaki spotkasz jako frontend developer. To nie protokół, lecz **zestaw zasad projektowania** wykorzystujący [HTTP](../01-fundamenty-sieci/02-protokol-http.md) zgodnie z jego semantyką. Znajomość REST to głównie umiejętność **konsumowania** API poprawnie i rozpoznawania, kiedy jest dobrze/źle zaprojektowane.

## Zasady REST

- **Zasoby** identyfikowane URL-em (rzeczowniki, nie czasowniki): `/users/42/orders`, nie `/getUserOrders?id=42`.
- **Metody HTTP niosą czasownik** ([metody](../01-fundamenty-sieci/02-protokol-http.md)): `GET` (odczyt), `POST` (utwórz), `PUT` (podmień), `PATCH` (zmień częściowo), `DELETE`.
- **Bezstanowość** — każdy request samodzielny (auth w każdym: [uwierzytelnianie](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)).
- **Statusy komunikują wynik** ([kody](../01-fundamenty-sieci/02-protokol-http.md)): 2xx sukces, 4xx błąd klienta, 5xx błąd serwera.
- „Poziomy dojrzałości" (Richardson) i HATEOAS to teoria — realne API są zwykle „pragmatycznym REST".

```
GET    /api/products?category=shoes&page=2   → lista (filtrowanie/paginacja)
GET    /api/products/42                        → jeden zasób
POST   /api/products        { … }              → 201 Created + Location
PATCH  /api/products/42     { price: 199 }      → 200 + zaktualizowany
DELETE /api/products/42                         → 204 No Content
```

## Wersjonowanie

API ewoluuje; zmiany łamiące (breaking) wymagają wersji, by nie zepsuć klientów:
- W ścieżce: `/api/v1/...` (najczęstsze, czytelne).
- W nagłówku: `Accept: application/vnd.api+json; version=1`.
- Zasada: **dodawanie pól nie łamie** (klient ignoruje nieznane), usuwanie/zmiana znaczenia — łamie. Projektuj klienta tolerancyjnie (nie zakładaj sztywno kształtu; [walidacja na granicy](../06-typescript/05-typescript-w-praktyce.md)).

## Paginacja, filtrowanie, sortowanie

- **Offset/limit** (`?page=2&limit=20` lub `?offset=40&limit=20`) — proste, ale niestabilne przy zmianach danych (duplikaty/pominięcia przy wstawieniach) i wolne dla głębokich stron.
- **Cursor-based** (`?cursor=abc&limit=20`) — stabilne i wydajne dla „infinite scroll"/dużych zbiorów; brak skoku do dowolnej strony. Preferowane w nowych API ([system design: infinite scroll](../17-rekrutacja/04-frontend-system-design.md)).
- Metadane w odpowiedzi (`total`, `nextCursor`) albo w nagłówku `Link`.
- Filtrowanie/sortowanie w query params; złożone filtry bywają problemem (stąd czasem [GraphQL](./02-graphql.md)).

## Obsługa błędów API — kontrakt

Dobre API zwraca **ustrukturyzowany błąd**, nie goły status:

```json
{ "error": { "code": "VALIDATION_ERROR",
             "message": "…",
             "fields": { "email": "Nieprawidłowy format" } } }
```

Standard, który warto znać/proponować: **RFC 9457 Problem Details** (`application/problem+json`). Po stronie klienta ([warstwa API](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)): sprawdź `res.ok`, zmapuj `4xx/5xx` na błędy domenowe ([strategia błędów](../04-javascript/11-obsluga-bledow.md)), błędy walidacji rozłóż na pola formularza ([formularze](../02-html/03-formularze.md)).

## Konsumowanie REST — praktyka frontendu

1. **Nie rozsiewaj `fetch`** — warstwa API/klient ([warstwa danych](../08-architektura-aplikacji/04-struktura-projektu.md)): bazowy URL, auth, błędy, retry, typy.
2. **Mapuj DTO → model domenowy** — nie pozwól, by kształt backendu przeciekał do całego UI (łatwiej przeżyć zmianę API — [adapter](../04-javascript/13-wzorce-projektowe-js.md)).
3. **Typuj/waliduj odpowiedzi** — codegen z OpenAPI albo schema (Zod) na granicy ([TS w praktyce](../06-typescript/05-typescript-w-praktyce.md)); typy „na słowo" backendu kłamią.
4. **Cache i stany** to osobny, duży temat: [wzorce pracy z danymi](./04-wzorce-pracy-z-danymi.md) i [cache](../09-wydajnosc/04-strategie-cachowania.md).
5. **N+1 z perspektywy klienta** — lista, a potem request per element = wodospad; potrzebujesz endpointu zwracającego komplet (albo GraphQL/batch).

## OpenAPI / Swagger

Maszynowy opis REST API (endpointy, kształty, statusy). Dla frontendu bezcenny: **codegen** typowanego klienta i typów, dokumentacja, mocki, kontrakt do [contract testing](../13-testowanie/04-testy-e2e-i-specjalistyczne.md). Jeśli backend ma OpenAPI — generuj klienta, nie pisz go ręcznie.

## Pułapki i częste błędy

- Czasowniki w URL i `GET` zmieniający stan ([metody — bezpieczeństwo/idempotencja](../01-fundamenty-sieci/02-protokol-http.md)).
- Zakładanie sztywnego kształtu odpowiedzi bez walidacji — crash po zmianie API.
- Kształt DTO rozlany po całym UI — każda zmiana backendu = refaktor wszędzie.
- Offset pagination dla „infinite scroll" na zmieniających się danych — duplikaty/luki.
- Traktowanie statusu 200 z `{ error: ... }` w body jako sukcesu (niektóre API tak robią — sprawdzaj kontrakt, nie tylko `res.ok`).
- Brak obsługi `429`/retry-after (rate limiting) — [wzorce pracy z danymi](./04-wzorce-pracy-z-danymi.md).

## Pytania rekrutacyjne

1. **Co czyni API RESTowym?** — zasoby+metody HTTP+bezstanowość+statusy; pragmatyzm vs czystość.
2. **PUT vs PATCH vs POST — idempotencja?** — podmiana/część/utworzenie; które idempotentne ([HTTP](../01-fundamenty-sieci/02-protokol-http.md)).
3. **Offset vs cursor pagination — kompromisy?** — skok do strony vs stabilność/wydajność dla dużych zbiorów.
4. **Jak zaprojektujesz warstwę konsumpcji API?** — klient, DTO→domena, typy/walidacja, błędy, cache.
5. **Jak wersjonujesz API i co jest zmianą łamiącą?** — ścieżka/nagłówek; dodawanie vs usuwanie/zmiana znaczenia.

## Dalsza lektura

- [MDN: HTTP overview (semantyka pod REST)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
- [OpenAPI Specification](https://swagger.io/specification/)
