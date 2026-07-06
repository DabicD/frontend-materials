# Domeny, hosting i CDN

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Jak działa internet](./01-jak-dziala-internet.md)

Ten plik domyka obraz infrastruktury: jak zbudowany jest URL, jak domena wskazuje na serwer (rekordy DNS) i jak CDN przybliża treści do użytkownika. Wiedza niezbędna przy deploymencie i debugowaniu problemów „u klienta działa, u nas nie".

## Anatomia URL

```
https://www.example.com:443/products/list?page=2&sort=asc#reviews
└─┬─┘   └──────┬───────┘└┬┘└─────┬──────┘└──────┬───────┘└──┬───┘
schemat      host       port   ścieżka     query string   fragment
```

- **Schemat (protokół):** `https`, `http`, `ws`, `mailto`…
- **Host:** subdomena (`www`) + domena (`example`) + TLD (`.com`). Całość czytana od prawej: TLD → domena → subdomeny.
- **Port:** domyślnie 443 (https) / 80 (http) — pomijany, gdy domyślny.
- **Ścieżka:** hierarchiczny adres zasobu.
- **Query string:** parametry `?klucz=wartość&...` — dostępne w JS przez `URLSearchParams`.
- **Fragment (hash):** `#reviews` — **nie jest wysyłany do serwera**; służy do kotwic na stronie i bywał używany do routingu SPA ([routing po stronie klienta](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)).

**Origin** = schemat + host + port. To fundamentalne pojęcie bezpieczeństwa (Same-Origin Policy, CORS) — dom: [mechanizmy obronne](../11-bezpieczenstwo/02-mechanizmy-obronne.md). `https://example.com` i `https://api.example.com` to **różne originy**.

Do pracy z URL-ami w JS służy klasa `URL`:

```js
const url = new URL('https://example.com/products?page=2');
url.searchParams.get('page'); // '2'
url.pathname;                 // '/products'
```

## Rekordy DNS, które musisz kojarzyć

| Rekord | Co robi | Przykład użycia |
|---|---|---|
| `A` / `AAAA` | domena → adres IPv4 / IPv6 | `example.com → 76.76.21.21` |
| `CNAME` | alias: domena → inna domena | `www.example.com → example.com`, `app.example.com → cname.vercel-dns.com` |
| `TXT` | dowolny tekst | weryfikacja własności domeny, SPF/DKIM |
| `MX` | serwery poczty | — |
| `NS` | serwery nazw (kto zarządza strefą) | delegacja do Cloudflare |

Praktyka frontendowca: podpinając domenę pod hosting (Vercel/Netlify/Cloudflare Pages), ustawiasz zwykle rekord `A` dla domeny głównej i `CNAME` dla subdomen. Pamiętaj o **propagacji DNS** — zmiany rozchodzą się do TTL starego rekordu.

## Rodzaje hostingu frontendu

- **Static hosting** — serwowanie gotowych plików (HTML/CSS/JS) z CDN. Idealne dla SPA i stron statycznych: tanie, szybkie, skalowalne (S3+CloudFront, Netlify, GitHub Pages).
- **Serwer aplikacyjny / SSR** — proces Node (lub edge runtime) renderujący HTML na żądanie; potrzebny dla [SSR/ISR](../08-architektura-aplikacji/01-wzorce-renderowania.md).
- **Edge computing** — kod uruchamiany w setkach lokalizacji CDN blisko użytkownika (Cloudflare Workers, Vercel Edge). Minimalna latencja, ale ograniczone runtime'y.

Szczegóły procesu wdrażania: [hosting i deployment](../16-devops-dla-frontendu/02-hosting-i-deployment.md).

## CDN (Content Delivery Network)

CDN to rozproszona sieć serwerów cache (**edge/PoP — points of presence**) między użytkownikiem a serwerem źródłowym (**origin server**).

**Jak działa:** użytkownik z Warszawy prosi o `app.js` → DNS kieruje go do najbliższego PoP (np. Warszawa/Frankfurt) → **cache HIT**: plik wraca w kilkanaście ms; **cache MISS**: PoP pobiera z origin, zapisuje u siebie i oddaje. Kolejni użytkownicy z regionu dostają kopię z edge.

**Co daje:**
1. **Latencja** — fizyczna bliskość = krótszy RTT (nie da się oszukać prędkości światła).
2. **Odciążenie origin** — 90%+ ruchu statycznego nie dociera do Twojego serwera.
3. **Odporność** — absorpcja skoków ruchu i ataków DDoS.

**Czym CDN steruje:** nagłówkami [HTTP cache](./02-protokol-http.md) (`Cache-Control: s-maxage` dotyczy właśnie cache współdzielonych) oraz **inwalidacją/purge** przy deploymencie. Pełny obraz warstw cache: [strategie cache'owania](../09-wydajnosc/04-strategie-cachowania.md).

## Pułapki i częste błędy

- Traktowanie subdomeny jako „tej samej strony" — to inny origin: osobny CORS, osobny `localStorage`, cookies tylko przy jawnym `Domain=`.
- Zapominanie o purge/inwalidacji CDN po deploymencie plików **bez** hasha w nazwie — użytkownicy widzą starą wersję do wygaśnięcia TTL.
- Testowanie tylko na łączu biurowym — CDN maskuje wolny origin dla treści cache'owalnych, ale nie dla API.
- Wrzucanie na CDN odpowiedzi personalizowanych bez `private`/`Vary` — ryzyko podania cudzych danych z cache współdzielonego.

## Pytania rekrutacyjne

1. **Z czego składa się URL i co to jest origin?** — schemat+host+port(+ścieżka+query+fragment); origin = schemat+host+port; fragment nie idzie do serwera.
2. **Czym różni się rekord A od CNAME?** — A wskazuje IP; CNAME wskazuje inną nazwę domenową (i nie może istnieć na apexie domeny obok innych rekordów).
3. **Jak CDN przyspiesza stronę i co się dzieje przy cache MISS?** — serwowanie z edge blisko użytkownika; przy MISS pobranie z origin i zapis w cache PoP.
4. **Dlaczego po deploymencie użytkownicy widzą starą wersję strony?** — cache CDN/przeglądarki: brak cache bustingu lub brak inwalidacji; rozwiązania: hash w nazwach plików, `no-cache` dla HTML, purge CDN.

## Dalsza lektura

- [MDN: What is a URL?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL)
- [Cloudflare Learning: What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)
- [web.dev: Content delivery networks](https://web.dev/articles/content-delivery-networks)
