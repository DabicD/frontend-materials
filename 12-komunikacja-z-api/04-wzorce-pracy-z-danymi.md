# Wzorce pracy z danymi (data fetching)

> **Poziom:** 🔴 zaawansowany (codzienna praca, częsty temat rozmów)
> **Wymagana wiedza:** [Fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md), [Asynchroniczność](../04-javascript/08-asynchronicznosc.md), [Zarządzanie stanem](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)

Pobranie danych to nie `await fetch()` — to **zarządzanie stanem asynchronicznym**: stany ładowania/błędu, cache, świeżość, race conditions, ponawianie, optymistyczne aktualizacje. Te problemy są wspólne dla każdej aplikacji; dlatego powstały biblioteki server-state (TanStack Query, SWR, RTK Query, klienci GraphQL). Ten plik to **katalog problemów i wzorców** niezależny od biblioteki.

## Modelowanie stanu żądania

Minimum to trzy stany — ale „trzy boolean" to pułapka; użyj [discriminated union](../06-typescript/02-interfejsy-i-typy.md):

```ts
type Query<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

Wyklucza to niemożliwe kombinacje (`loading && error`) i wymusza obsłużenie każdego stanu w UI ([model komponentowy — granice](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)). Dochodzą stany subtelniejsze: `refetching` (mam dane, odświeżam), `stale` (dane nieświeże), empty (sukces, ale pusta lista — inny UI niż loading!).

## Race conditions — problem, o który pytają

Użytkownik szybko zmienia zapytanie (wyszukiwarka, przełączanie zakładek). Requesty wracają **w innej kolejności niż wysłane** → wynik starego zapytania nadpisuje nowy:

```
zapytanie "ab"  ──────────────▶ (wolne) wraca DRUGIE
zapytanie "abc" ────▶ (szybkie) wraca PIERWSZE, potem nadpisane przez "ab" (!)
```

Rozwiązania:
- **Anuluj poprzedni** — [AbortController](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md) przy nowym zapytaniu (najczystsze).
- **Ignoruj nieaktualne** — zapamiętaj „najświeższy request id", odrzuć odpowiedzi starsze.
- **Debounce** wejścia ogranicza liczbę requestów ([optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)), ale **nie usuwa** wyścigu — łącz z anulowaniem.
- Biblioteki query robią to za Ciebie (klucz zapytania = tożsamość; stare odrzucane).

## Cache i świeżość (server state)

Dane z API to **cache cudzego stanu** — nie „zmienna" ([taksonomia stanu](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)). Mechanizmy, które musisz mieć (i które biblioteki dają):

- **Deduplikacja** — dwa komponenty proszą o to samo → jeden request.
- **Stale-while-revalidate** — pokaż z cache natychmiast, odśwież w tle ([strategie cache'owania](../09-wydajnosc/04-strategie-cachowania.md)).
- **Klucz zapytania** — identyfikuje dane (`['user', id]`); zmiana klucza = nowe dane.
- **Inwalidacja po mutacji** — po zapisie unieważnij powiązane zapytania (lista po dodaniu elementu).
- **Refetch triggers** — focus okna, reconnect, interwał.
- **Garbage collection** cache'u nieużywanych zapytań.

Ręczne odtworzenie tego w [globalnym store](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md) = przepisywanie biblioteki; stąd „nie trzymaj server state w Reduxie".

## Optimistic updates

Dla akcji, które „prawie zawsze się udają" (polubienie, dodanie do koszyka) — zaktualizuj UI **natychmiast**, przed odpowiedzią serwera; przy błędzie **cofnij (rollback)**:

```
klik → od razu pokaż "polubione" (zapamiętaj poprzedni stan)
     → wyślij request
     → sukces: zostaw / potwierdź serwerowym stanem
     → błąd: rollback + komunikat
```

Daje responsywność (INP — [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md)); wymaga dyscypliny rollbacku i obsługi konfliktów.

## Retry i odporność

- **Retry z exponential backoff + jitter** dla błędów przejściowych (sieć, `5xx`, `429`); **nie** retry `4xx` (poza `408`/`429`) — to nie naprawi błędu klienta.
- Retry tylko operacji **idempotentnych** albo z kluczem idempotencji ([HTTP — idempotencja](../01-fundamenty-sieci/02-protokol-http.md)) — inaczej podwójne zamówienia.
- Respektuj `Retry-After` (rate limiting).
- Timeouty (`AbortSignal.timeout`) — nie każ użytkownikowi czekać w nieskończoność.
- Mapowanie błędów na komunikaty domenowe ([strategia błędów](../04-javascript/11-obsluga-bledow.md)).

## UX stanów asynchronicznych

- **Skeletony/placeholdery** o wymiarach docelowej treści (przeciw [CLS](../09-wydajnosc/01-core-web-vitals.md)); spinner dopiero po progu (~200–300 ms), by uniknąć migotania.
- **Rozróżniaj empty od loading od error** — inne komunikaty, inne akcje (retry vs „dodaj pierwszy element").
- **Zachowaj poprzednie dane** podczas refetchu (paginacja: nie migaj do pustego).
- **Prefetch** przewidywalnych danych (hover na linku, następna strona) — [routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md), [ładowanie](../09-wydajnosc/02-optymalizacja-ladowania.md).
- Waterfall vs równoległość: startuj niezależne zapytania razem ([Promise.all](../04-javascript/08-asynchronicznosc.md)); dane trasy ładuj z nawigacją, nie po hydracji.

## Pułapki i częste błędy

- „loading boolean" i brak obsługi błędu/empty — białe ekrany i zawieszone spinnery.
- Ignorowanie race conditions w wyszukiwarce/filtrach — miganie i złe wyniki.
- Server state w globalnym store z ręczną inwalidacją — bugi cache i duplikaty.
- `useEffect`/fetch w komponencie bez anulowania i cleanup — leaki i wyścigi po unmount.
- Retry wszystkiego (w tym `POST` i `4xx`) — podwojone efekty, maskowanie realnych błędów.
- Optimistic update bez rollbacku — UI kłamie po nieudanej akcji.
- Brak deduplikacji — N komponentów = N identycznych requestów.

## Pytania rekrutacyjne

1. **Jakie stany ma zapytanie i jak je modelujesz?** — idle/loading/success/error(+refetching/empty); unia zamiast booleanów.
2. **Co to race condition w data fetchingu i jak go rozwiązujesz?** — kolejność odpowiedzi; abort/ignore stale/debounce.
3. **Czemu server state nie należy do zwykłego store'a?** — cache cudzego stanu: świeżość, dedup, invalidacja; biblioteki query.
4. **Jak działają optimistic updates i jaki mają koszt?** — natychmiastowy UI + rollback; konflikty.
5. **Zasady retry?** — backoff+jitter, tylko przejściowe/idempotentne, Retry-After, nie `4xx`.
6. **Jak zaprojektujesz UX ładowania listy z paginacją?** — skeleton→dane, zachowanie poprzednich, empty≠loading, prefetch.

## Dalsza lektura

- [TanStack Query — dokumentacja i przewodniki](https://tanstack.com/query/latest/docs/framework/react/overview) (koncepty przenośne)
- [SWR docs](https://swr.vercel.app/)
- [web.dev / Kent C. Dodds — o stanach UI i race conditions](https://kentcdodds.com/blog/stop-using-isloading-booleans)
