# Mechanizmy obronne: SOP, CORS, CSP, nagłówki

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Ataki na frontend](./01-ataki-na-frontend.md), [Protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md), [Origin](../01-fundamenty-sieci/04-domeny-hosting-cdn.md)

Przeglądarka ma wbudowane mechanizmy bezpieczeństwa; Twoim zadaniem jest je **rozumieć i konfigurować**. Fundamentem jest Same-Origin Policy; CORS ją kontrolowanie rozluźnia; CSP, SRI i nagłówki bezpieczeństwa domykają obronę. To materiał, na którym poznaje się seniora — zwłaszcza CORS, źle rozumiany przez większość.

## Same-Origin Policy (SOP)

Domyślna zasada: skrypt z jednego [originu](../01-fundamenty-sieci/04-domeny-hosting-cdn.md) (schemat+host+port) **nie może czytać** danych z innego originu. To fundament izolacji: gdyby nie SOP, skrypt na złej stronie czytałby Twoją pocztę z otwartej karty.

- Blokowany jest **odczyt** odpowiedzi cross-origin, nie samo wysłanie.
- Co SOP **przepuszcza** (osadzanie): `<img>`, `<script>`, `<link>`, `<iframe>`, formularze — mogą wskazywać obce originy (stąd możliwość [CSRF](./01-ataki-na-frontend.md) i cały ekosystem CDN).
- Rozluźnienia: CORS (dla `fetch`/XHR), `postMessage` (kontrolowana komunikacja między oknami — **zawsze sprawdzaj `event.origin`!**).

## CORS — najczęściej źle rozumiany

CORS (Cross-Origin Resource Sharing) to sposób, w jaki **serwer zezwala** wybranym originom na czytanie swoich odpowiedzi. Kluczowe fakty:

1. **CORS to zgoda serwera, nie zabezpieczenie Twojej aplikacji.** Nagłówki ustawia **serwer docelowy**; Ty jako frontend nic w nich nie zmienisz.
2. **Błąd CORS = brak zgody serwera**, nie bug w Twoim kodzie. „Naprawiasz" go po stronie serwera API (albo przez proxy), nie w przeglądarce.
3. To **przeglądarka** egzekwuje CORS — `curl`/backend nie mają tego ograniczenia (dlatego „w Postmanie działa").

**Requesty proste vs preflight:**
- **Proste** (GET/POST/HEAD z „bezpiecznymi" nagłówkami, np. formularzowy Content-Type) — lecą od razu; przeglądarka sprawdza `Access-Control-Allow-Origin` w odpowiedzi i **blokuje odczyt**, jeśli origin niedozwolony.
- **Preflight** — dla „nietrywialnych" (metody PUT/DELETE/PATCH, `Content-Type: application/json`, custom headers, `Authorization`) przeglądarka wysyła najpierw `OPTIONS` ([metoda](../01-fundamenty-sieci/02-protokol-http.md)) i czeka na zgodę:

```http
> OPTIONS /api  (Origin, Access-Control-Request-Method, -Headers)
< Access-Control-Allow-Origin: https://app.example.com
< Access-Control-Allow-Methods: GET, POST, PUT, DELETE
< Access-Control-Allow-Headers: Content-Type, Authorization
< Access-Control-Max-Age: 86400        # cache preflightu
```

- **Cookies cross-origin:** `credentials: 'include'` (klient) **+** `Access-Control-Allow-Credentials: true` **+** konkretny origin (nie `*`) po stronie serwera ([fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md), [uwierzytelnianie](./03-uwierzytelnianie-i-autoryzacja.md)).

## CSP (Content Security Policy) — druga linia obrony przed XSS

Nagłówek (lub meta) mówiący przeglądarce, **skąd wolno ładować zasoby i co wykonywać**. Nawet jeśli atakujący wstrzyknie skrypt, CSP może go zablokować:

```http
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-r4nd0m';       # tylko własne + skrypty z tym nonce
  style-src 'self';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';                  # anty-clickjacking (zastępuje X-Frame-Options)
  object-src 'none'; base-uri 'self';
```

- **Cel:** wyeliminować inline scripts i `eval` — najskuteczniejsza forma to **nonce/hash** zamiast `'unsafe-inline'` (które niweczy sens CSP). Nowocześnie: **strict-dynamic** + nonce.
- CSP to **defense in depth** — uzupełnia, nie zastępuje escapingu/sanityzacji ([ataki](./01-ataki-na-frontend.md)).
- Wdrażaj przez `Content-Security-Policy-Report-Only` + endpoint raportujący → iteruj bez psucia produkcji.
- `frame-ancestors` chroni przed [clickjackingiem](./01-ataki-na-frontend.md); `connect-src` ogranicza eksfiltrację danych.

## SRI (Subresource Integrity)

Dla skryptów/styli z CDN — hash gwarantujący, że plik się nie zmienił (obrona przed [supply chain](./01-ataki-na-frontend.md) na poziomie CDN):

```html
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-…" crossorigin="anonymous"></script>
```

Przeglądarka policzy hash i **odmówi wykonania** przy niezgodności. Buildy generują SRI dla własnych zasobów automatycznie.

## Nagłówki bezpieczeństwa — zestaw bazowy

| Nagłówek | Chroni przed | Wartość |
|---|---|---|
| `Content-Security-Policy` | XSS, injection, eksfiltracja | jak wyżej |
| `Strict-Transport-Security` | downgrade HTTPS→HTTP | `max-age=63072000; includeSubDomains; preload` |
| `X-Content-Type-Options` | MIME sniffing | `nosniff` |
| `X-Frame-Options` / CSP `frame-ancestors` | clickjacking | `DENY` |
| `Referrer-Policy` | wyciek URL w `Referer` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | nadużycie API (kamera, geo) | wyłącz nieużywane |

Nowoczesna izolacja: **COOP/COEP/CORP** (cross-origin isolation) — wymagane m.in. dla `SharedArrayBuffer` ([workery](../05-dom-i-web-api/06-web-workers.md)); dają silniejszą separację originów. Nagłówki ustawia się na serwerze/edge/hostingu ([hosting](../16-devops-dla-frontendu/02-hosting-i-deployment.md)); audyt: securityheaders.com, Mozilla Observatory.

## Pułapki i częste błędy

- „Naprawianie CORS" w kodzie frontendu (dodawanie nagłówków `Access-Control-*` do **requestu**) — to robi serwer; klient tylko wysyła `Origin`.
- `mode: 'no-cors'` jako obejście — dostajesz opaque response bez dostępu do danych ([fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).
- CSP z `'unsafe-inline'`/`'unsafe-eval'` „żeby działało" — otwiera to, przed czym miał chronić.
- `Access-Control-Allow-Origin: *` **razem** z credentials — zabronione; wybierz konkretny origin.
- Brak HSTS — podatność na SSL stripping przy pierwszym połączeniu.
- Traktowanie nagłówków jako jednorazowej konfiguracji — audytuj po zmianach infrastruktury.

## Pytania rekrutacyjne

1. **Wyjaśnij Same-Origin Policy i co przepuszcza.** — izolacja odczytu; osadzanie zasobów dozwolone.
2. **Czym jest CORS i kto go konfiguruje? Dostajesz błąd CORS — co robisz?** — zgoda serwera; naprawa po stronie API/proxy, nie klienta.
3. **Kiedy leci preflight i po co?** — nietrywialne requesty; OPTIONS pyta o zgodę przed właściwym.
4. **Jak CSP chroni przed XSS i jak ją poprawnie wdrożyć?** — ograniczenie źródeł, nonce/hash zamiast unsafe-inline, report-only najpierw.
5. **Wymień kluczowe nagłówki bezpieczeństwa.** — CSP, HSTS, nosniff, frame-ancestors, Referrer-Policy, Permissions-Policy.

## Dalsza lektura

- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) i [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)
- [web.dev: Security headers / Secure your site](https://web.dev/articles/security-headers)
- [securityheaders.com](https://securityheaders.com/) + [Mozilla Observatory](https://developer.mozilla.org/en-US/observatory)
