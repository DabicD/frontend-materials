# Strategie cache'owania — pełny obraz warstw

> **Poziom:** 🔴 zaawansowany (synteza)
> **Wymagana wiedza:** [HTTP caching](../01-fundamenty-sieci/02-protokol-http.md) (mechanika nagłówków — tam jest dom), [CDN](../01-fundamenty-sieci/04-domeny-hosting-cdn.md), [Service Worker](../08-architektura-aplikacji/02-spa-mpa-pwa.md)

Cache to najskuteczniejsza optymalizacja: najszybszy request to ten, którego nie ma. Ten plik **skleja warstwy w jedną strategię** — mechanika poszczególnych ogniw jest w plikach-domach (linki powyżej i w tekście).

## Mapa warstw — którędy podróżuje request

```
JS (pamięć: query cache) → Service Worker (Cache API) → HTTP cache przeglądarki
   → CDN edge → cache serwera aplikacji → origin/baza
```

Każda warstwa: inny zasięg (użytkownik/urządzenie/region/wszyscy), inny czas życia, inny mechanizm inwalidacji. Projektowanie cache = decyzja **per typ zasobu**, nie globalna.

## Strategia per typ zasobu (kanoniczny zestaw)

| Zasób | Strategia | Realizacja |
|---|---|---|
| Statyki z hashem (`app.3f2a.js`) | cache „wieczny", immutable | `max-age=31536000, immutable` ([mechanika](../01-fundamenty-sieci/02-protokol-http.md)) + CDN |
| HTML | zawsze rewaliduj | `no-cache` (+ CDN `s-maxage` z purge przy deployu) |
| API — dane wspólne (katalog) | krótki TTL współdzielony + SWR | `s-maxage=60, stale-while-revalidate=600` na CDN |
| API — dane usera | prywatnie albo wcale | `private, no-store`/krótki `private, max-age` |
| Obrazy/fonty | długo, wersjonowane URL-e | jak statyki |
| Offline/PWA | wg krytyczności | Service Worker: strategie niżej |

**Cache busting** — fundament całości: hash treści w nazwie pliku ([build](../14-narzedzia/03-bundlery-i-proces-budowania.md)) zamienia problem inwalidacji na problem nazewnictwa — nowa treść = nowy URL, stara może żyć w cache wiecznie.

## Stale-While-Revalidate — wzorzec, który spina wszystko

Serwuj **natychmiast** z cache (nawet przeterminowane), **w tle** odśwież na następny raz:

- HTTP: rozszerzenie `Cache-Control: stale-while-revalidate=…` (CDN).
- Service Worker: strategia SWR w [Workbox](../08-architektura-aplikacji/02-spa-mpa-pwa.md).
- Warstwa JS: dokładnie ten model implementują biblioteki server-state (SWR — nomen omen, TanStack Query): pokaż z cache → refetch w tle → podmień ([wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)).
- [ISR](../08-architektura-aplikacji/01-wzorce-renderowania.md) to SWR na poziomie całych stron.

Jeden model mentalny na czterech piętrach stacka — na rozmowie robi wrażenie, bo pokazuje syntezę.

## Inwalidacja — najtrudniejszy problem

1. **Wersjonowanie URL-i** (hash) — inwalidacja przez zmianę adresu; domyślne dla statyk.
2. **TTL** — kompromis świeżość/koszt; krótkie dla danych, długie dla statyk.
3. **Purge na zdarzenie** — API CDN przy deployu/zmianie treści (webhook z CMS); precyzyjne tagi cache (surrogate keys).
4. **Rewalidacja warunkowa** — ETag/304 ([mechanika](../01-fundamenty-sieci/02-protokol-http.md)): oszczędza transfer, nie RTT.
5. W warstwie JS: inwalidacja **po mutacji** (zapis → unieważnij powiązane zapytania) — [wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md).

Zasada: **im wyżej (bliżej użytkownika) cache'ujesz, tym szybciej działa i tym trudniej unieważnić.** Dlatego dane personalizowane trzymaj nisko (JS/pamięć), wspólne — wysoko (CDN).

## Nagłówek `Vary` i klucze cache

Cache współdzielony indeksuje po URL — jeśli odpowiedź zależy od nagłówka (`Accept-Language`, `Accept-Encoding`), zadeklaruj `Vary`, inaczej użytkownicy dostaną cudze warianty. Zbyt szerokie `Vary` (np. na cookies) = cache praktycznie wyłączony. Personalizację lepiej wynieść do JS/edge zamiast różnicować cache'owane HTML-e.

## Pułapki i częste błędy

- Cache'owanie odpowiedzi z danymi usera na CDN (brak `private`) — wyciek danych między użytkownikami; incydent klasy poważnej.
- HTML z długim `max-age` — użytkownicy uwięzieni w starej wersji wskazującej nieistniejące chunki (błędy po deployu).
- Service Worker agresywnie cache'ujący własny plik/HTML ([pułapki SW](../08-architektura-aplikacji/02-spa-mpa-pwa.md)).
- Podwójne/rozjazdowe TTL (CDN dłużej niż przeglądarka myśli) — debuguj nagłówkami odpowiedzi (`age`, `x-cache`).
- Brak jakiejkolwiek strategii dla API — każdy klik = pełny round-trip, „wolna aplikacja" mimo szybkiego frontendu.
- Inwalidacja „wyczyść wszystko" przy każdej zmianie — zimny cache, lawina na origin (cache stampede; chroni `stale-while-revalidate`).

## Pytania rekrutacyjne

1. **Wymień warstwy cache między użytkownikiem a bazą.** — pamięć JS/SW/HTTP/CDN/serwer; właściwości każdej.
2. **Jak skonfigurujesz cache dla SPA po buildzie?** — hash+immutable dla statyk, no-cache HTML, purge CDN.
3. **Wyjaśnij stale-while-revalidate i gdzie występuje.** — natychmiast z cache + odświeżenie w tle; HTTP/SW/query-cache/ISR.
4. **Jak inwalidujesz cache po publikacji treści z CMS?** — webhook → purge po tagach; ISR on-demand.
5. **Do czego służy `Vary` i czym grozi jego brak/nadmiar?** — klucz cache po nagłówkach; cudze warianty vs martwy cache.

## Dalsza lektura

- [web.dev: HTTP cache + Love your cache](https://web.dev/articles/http-cache)
- [Cloudflare: cache directives i SWR](https://developers.cloudflare.com/cache/)
- [TanStack Query: Caching](https://tanstack.com/query/latest/docs/framework/react/guides/caching)
