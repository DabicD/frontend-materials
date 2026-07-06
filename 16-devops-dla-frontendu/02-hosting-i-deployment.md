# Hosting i deployment

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Domeny, hosting, CDN](../01-fundamenty-sieci/04-domeny-hosting-cdn.md), [Wzorce renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md), [CI/CD](./01-ci-cd.md)

Gdzie i jak żyje Twój frontend w produkcji. Wybór hostingu wynika bezpośrednio z [wzorca renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md): statyki idą na CDN, SSR potrzebuje runtime'u. Frontend developer powinien rozumieć te modele, zmienne środowiskowe i podstawy konteneryzacji — nawet jeśli platforma (Vercel/Netlify) ukrywa większość.

## Modele hostingu

**Static hosting + CDN** — gotowe pliki (HTML/CSS/JS) serwowane z [edge](../01-fundamenty-sieci/04-domeny-hosting-cdn.md):
- Dla [CSR/SSG](../08-architektura-aplikacji/01-wzorce-renderowania.md): najtańsze, najszybsze, najbezpieczniejsze (brak serwera do zhakowania), skalowalne z natury.
- Vercel, Netlify, Cloudflare Pages, GitHub Pages, AWS S3+CloudFront.
- **Wymóg SPA:** catch-all rewrite wszystkich ścieżek do `index.html` (inaczej 404 przy odświeżeniu głębokiego linku — [routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)).

**Serwer aplikacyjny (SSR)** — proces Node renderujący na żądanie:
- Dla [SSR/ISR](../08-architektura-aplikacji/01-wzorce-renderowania.md); potrzebny runtime, skalowanie, koszt, monitoring [TTFB](../09-wydajnosc/01-core-web-vitals.md).
- Platformy (Vercel/Netlify z serverless functions) lub własny hosting/kontenery.

**Edge computing** — kod w runtime CDN blisko użytkownika (Cloudflare Workers, Vercel Edge Functions):
- Minimalna latencja, SSR/personalizacja na brzegu; ograniczone runtime'y (nie pełny Node), limity CPU/czasu.

**Serverless functions** — endpointy backendowe „na żądanie" (BFF, [proxy do API](../14-narzedzia/03-bundlery-i-proces-budowania.md), obsługa formularzy, webhooki) bez zarządzania serwerem; uwaga na cold start.

## Proces deploymentu (git-based)

Nowoczesny standard: **push → automatyczny build i deploy**:

```
git push → webhook → platforma: npm ci + build → deploy artefaktu na CDN/runtime
         → preview URL (dla PR) / produkcja (dla main)
```

- **Immutable deployments** — każdy deploy to nowy, wersjonowany artefakt; poprzednie żyją → **rollback = przełączenie** na stary ([CI/CD](./01-ci-cd.md)).
- **Atomic deploys** — użytkownicy widzą albo całą starą, albo całą nową wersję (nie mieszankę w trakcie).
- **Preview deployments** per PR — [omówione w CI/CD](./01-ci-cd.md).

## Zmienne środowiskowe i konfiguracja

- Konfiguracja per środowisko (dev/staging/prod): URL API, klucze publiczne, feature flags.
- **Krytyczne dla bezpieczeństwa:** zmienne wbudowane w [bundle](../14-narzedzia/03-bundlery-i-proces-budowania.md) są **jawne** (DevTools → Sources). Tylko prefiksowane (`VITE_`/`NEXT_PUBLIC_`) trafiają do klienta; **sekrety** (klucze prywatne, tokeny) — wyłącznie po stronie serwera/funkcji ([ataki](../11-bezpieczenstwo/01-ataki-na-frontend.md)).
- **`.env`** lokalnie (nie commitować — [git](../14-narzedzia/01-git.md) `.gitignore`); w produkcji — panel platformy / secret manager.
- Build per środowisko albo runtime config (dla SSR) — inny zestaw zmiennych.

## Nagłówki i domena

- **Nagłówki bezpieczeństwa** ([CSP, HSTS...](../11-bezpieczenstwo/02-mechanizmy-obronne.md)) i **cache** ([Cache-Control](../09-wydajnosc/04-strategie-cachowania.md)) ustawia się na hostingu/edge (pliki konfiguracyjne platformy: `_headers`, `vercel.json` itp.).
- **HTTPS** — dziś domyślnie i automatycznie (Let's Encrypt); wymóg większości [API przeglądarki](../05-dom-i-web-api/07-przydatne-api-przegladarki.md).
- Podpięcie [domeny](../01-fundamenty-sieci/04-domeny-hosting-cdn.md) — rekordy A/CNAME; przekierowanie apex↔www.

## Konteneryzacja — podstawy (świadomość)

Choć frontend rzadko tego wymaga, warto rozumieć:

- **Docker** — pakuje aplikację + zależności + runtime w powtarzalny obraz („działa wszędzie tak samo"). Dla frontendu: zwykle **multi-stage build** (etap build z Node → skopiuj statyki do lekkiego obrazu Nginx/serwera statycznego).
- **Kontener vs VM** — kontener współdzieli kernel hosta (lekki, szybki start) vs pełna maszyna wirtualna.
- **Orkiestracja** (Kubernetes) — zarządzanie wieloma kontenerami; domena głównie backend/platform, ale frontend SSR bywa w nią wpięty.
- Kiedy potrzebne: własna infrastruktura, SSR poza platformami PaaS, spójność ze stackiem backendu, wymogi compliance.

## Pułapki i częste błędy

- Brak catch-all rewrite dla SPA → 404 po odświeżeniu podstron.
- Sekrety w zmiennych frontendu (prefiksowanych/wbudowanych) — jawne w bundlu.
- Niespójna [konfiguracja cache](../09-wydajnosc/04-strategie-cachowania.md) → stara wersja/błędy chunków po deployu.
- Brak rollbacku / mutowalne deploye — trudny powrót do działającej wersji.
- Ten sam build na wszystkie środowiska bez rozróżnienia konfiguracji (staging bije w produkcyjne API).
- `.env` z sekretami w repozytorium.
- Deploy statyki bez [nagłówków bezpieczeństwa](../11-bezpieczenstwo/02-mechanizmy-obronne.md).

## Pytania rekrutacyjne

1. **Jak wybierasz hosting dla aplikacji?** — od wzorca renderowania: statyka+CDN vs SSR-runtime vs edge.
2. **Czemu SPA daje 404 po odświeżeniu i jak to naprawić?** — brak tras na serwerze; rewrite do index.html.
3. **Gdzie mogą, a gdzie nie mogą trafić zmienne środowiskowe?** — publiczne w bundlu (jawne), sekrety tylko po stronie serwera.
4. **Co to immutable/atomic deployment i po co?** — wersjonowane artefakty, spójna wersja; łatwy rollback.
5. **Kiedy sięgniesz po Docker w projekcie frontendowym?** — własna infra, SSR, spójność z backendem; multi-stage build.

## Dalsza lektura

- [Vercel](https://vercel.com/docs) / [Netlify](https://docs.netlify.com/) / [Cloudflare Pages](https://developers.cloudflare.com/pages/) — docs
- [Docker: Get started](https://docs.docker.com/get-started/)
- [MDN: Setting up a deployment / hosting](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Server-side/First_steps)
