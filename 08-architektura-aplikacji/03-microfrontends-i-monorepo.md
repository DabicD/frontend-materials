# Microfrontends i monorepo

> **Poziom:** 🔴 zaawansowany (architektura organizacji, tematy rozmów senior+)
> **Wymagana wiedza:** [Moduły](../04-javascript/10-moduly.md), [Bundlery](../14-narzedzia/03-bundlery-i-proces-budowania.md), [Struktura projektu](./04-struktura-projektu.md)

Dwa tematy o **skalowaniu pracy wielu zespołów nad frontendem** — często mylone, bo oba dotyczą „dzielenia kodu". Monorepo dzieli **repozytorium**; microfrontends dzielą **aplikację w runtime**. Prawo Conwaya w praktyce: architektura odzwierciedla strukturę organizacji.

## Monorepo

Jedno repozytorium, wiele pakietów/aplikacji:

```
repo/
├── apps/        # aplikacje deployowalne (shop, admin, docs)
├── packages/    # współdzielone: ui (design system), utils, config, api-client
└── package.json # workspaces
```

- **Workspaces** (npm/pnpm/yarn) — pakiety linkowane lokalnie; `import { Button } from '@acme/ui'` bez publikowania do rejestru ([pakiety](../14-narzedzia/02-pakiety-i-zaleznosci.md)).
- **Narzędzia orkiestracji** — Turborepo/Nx: graf zależności między pakietami, **cache buildów/testów** i uruchamianie tylko tego, co dotknięte zmianą (affected) — bez tego CI monorepo umiera ([CI/CD](../16-devops-dla-frontendu/01-ci-cd.md)).
- ✚ atomowe zmiany cross-package (refaktor design systemu + aplikacji w 1 PR), jedna wersja zależności, łatwe współdzielenie, spójne narzędzia.
- ✖ rosnący czas CI (bez cache), sprzężenie zespołów (wspólny main), wymaga inwestycji w tooling.
- **Polyrepo** (osobne repa + pakiety npm): pełna autonomia zespołów; koszt: wersjonowanie, propagacja zmian przez publish/update, dryf wersji.

Monorepo ≠ monolit: deployujesz aplikacje niezależnie; to tylko strategia organizacji kodu.

## Microfrontends (MFE)

Aplikacja składana w **runtime** z niezależnie budowanych i deployowanych części, zwykle per zespół/domenę (checkout, katalog, konto):

**Techniki integracji:**
- **Module Federation** (webpack/Rspack; odpowiedniki w Vite) — aplikacje wymieniają się modułami w runtime; host ładuje `remoteEntry.js` zdalnych, współdzieląc biblioteki (singleton react itd.).
- **Web Components** — każda część jako custom element; naturalna izolacja ([Shadow DOM](../03-css/09-architektura-css.md)), framework-agnostic.
- **iframes** — najtwardsza izolacja (styl, JS, [bezpieczeństwo](../11-bezpieczenstwo/02-mechanizmy-obronne.md)), koszty UX (focus, resize, komunikacja postMessage).
- **Kompozycja na serwerze/edge** (ESI, streaming) lub **route-based** — najprostszy wariant: każda ścieżka to inna aplikacja, wspólny tylko nagłówek.

**Wspólne problemy do rozwiązania (o to pytają):**
- **Duplikacja zależności** — 3 kopie frameworka w bundle'u; Module Federation shared/singletons, import maps.
- **Spójność UX** — wspólny design system jako pakiet ([tokeny](../03-css/09-architektura-css.md)).
- **Komunikacja między MFE** — minimalizuj; URL, custom events, wąskie API — nie wspólny store (sprzężenie wraca tylnymi drzwiami).
- **Wersjonowanie kontraktów** między host a remote'ami; testy integracyjne granic ([contract testing](../13-testowanie/04-testy-e2e-i-specjalistyczne.md)).
- Routing, auth i telemetria przecinają wszystko — wymagają wspólnej „platformy".

## Decyzja — brutalna szczerość

MFE rozwiązują problem **organizacyjny** (autonomia deployów wielu zespołów na jednej stronie), nie techniczny. Heurystyka:

- 1–2 zespoły, jeden produkt → **modularny monolit w monorepo** (granice przez [strukturę modułów](./04-struktura-projektu.md)) — 90% korzyści, ułamek kosztów.
- Wiele zespołów, niezależne cykle wydań, być może różne stacki (fuzje firm) → MFE mają sens.
- Sygnał ostrzegawczy na rozmowie: proponowanie MFE „bo skalowalność" bez problemu organizacyjnego w tle.

## Pułapki i częste błędy

- MFE w małym zespole — cała złożoność, zero korzyści.
- Wspólny mutowalny stan między MFE — rozproszone sprzężenie gorsze niż monolit.
- Brak strategii wspólnych zależności — 4 MB frameworków.
- Monorepo bez cache/affected w CI — godzinne pipeline'y.
- Wydzielanie „packages" bez właściciela i kontraktu — wspólny śmietnik zamiast biblioteki.
- Mylenie granic technicznych (components/, utils/) z domenowymi (checkout/, catalog/) — [struktura projektu](./04-struktura-projektu.md).

## Pytania rekrutacyjne

1. **Monorepo vs polyrepo — kompromisy?** — atomowość i spójność vs autonomia; rola toolingu (cache, affected).
2. **Czym są microfrontends i jaki problem NAPRAWDĘ rozwiązują?** — niezależne deploye zespołów; Conway.
3. **Jak działa Module Federation?** — wymiana modułów w runtime, shared dependencies, host/remote.
4. **Jak zapewnisz spójność UX między MFE?** — design system, tokeny, wspólna platforma (auth/routing/telemetria).
5. **Kiedy odradzisz microfrontends?** — mały zespół/jeden produkt; alternatywa: modularny monolit.

## Dalsza lektura

- [Micro Frontends (martinfowler.com / Cam Jackson)](https://martinfowler.com/articles/micro-frontends.html)
- [Module Federation docs](https://module-federation.io/)
- [Turborepo](https://turborepo.com/docs) / [Nx](https://nx.dev/getting-started/intro)
