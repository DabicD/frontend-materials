# CI/CD dla frontendu

> **Poziom:** 🟡 średni → 🔴 (strategie wdrożeń)
> **Wymagana wiedza:** [Git](../14-narzedzia/01-git.md), [Testowanie](../13-testowanie/01-strategia-testowania.md), [Bundlery](../14-narzedzia/03-bundlery-i-proces-budowania.md)

CI/CD automatyzuje drogę od commita do produkcji — łapie błędy wcześnie i pozwala wdrażać **często i bezpiecznie**. Frontend developer nie musi być DevOps-em, ale musi rozumieć pipeline: co się dzieje z jego kodem po `git push` i czemu „zielony CI" to warunek merge'a.

## CI vs CD

- **Continuous Integration** — każdy commit/PR automatycznie: instaluje, buduje, sprawdza (lint, typy, testy). Cel: wykryć problemy integracji **wcześnie**, utrzymać `main` zawsze zdatny do wydania.
- **Continuous Delivery** — każda zmiana przechodząca CI jest **gotowa** do wdrożenia (deploy jednym kliknięciem/zatwierdzeniem).
- **Continuous Deployment** — automatyczny deploy na produkcję po zielonym pipeline, bez ręcznej bramki (wymaga dojrzałych testów, feature flags, monitoringu).

## Anatomia pipeline frontendowego

Typowy przepływ na PR (np. GitHub Actions):

```yaml
# .github/workflows/ci.yml (szkic)
on: [pull_request]
jobs:
  quality:
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4        # + cache zależności
      - run: npm ci                        # DOKŁADNIE z lockfile ([pakiety])
      - run: npm run lint                  # ESLint ([jakość kodu])
      - run: npm run typecheck             # tsc --noEmit ([TS w praktyce])
      - run: npm run test                  # unit + integracyjne ([testowanie])
      - run: npm run build                 # czy w ogóle się buduje
      - run: npm run test:e2e              # Playwright (często osobno/nocnie)
```

Etapy bramkujące merge (kolejność wg szybkości/kosztu — fail fast):
1. **Lint + format check** ([jakość kodu](../14-narzedzia/04-jakosc-kodu.md)) — sekundy.
2. **Type check** — `tsc --noEmit` (bundler nie sprawdza typów! — [TS](../06-typescript/05-typescript-w-praktyce.md)).
3. **Testy** unit/integracyjne ([testowanie](../13-testowanie/01-strategia-testowania.md)); [e2e](../13-testowanie/04-testy-e2e-i-specjalistyczne.md) wolniejsze — często równolegle/na osobnym triggerze.
4. **Build** — czy produkcyjny build przechodzi.
5. **Bramki jakości:** [Lighthouse CI](../09-wydajnosc/01-core-web-vitals.md) (budżety CWV), budżet wielkości [bundle'a](../14-narzedzia/03-bundlery-i-proces-budowania.md), [axe](../13-testowanie/04-testy-e2e-i-specjalistyczne.md) (a11y), skan sekretów/zależności ([bezpieczeństwo](../11-bezpieczenstwo/01-ataki-na-frontend.md)).

**Szybkość CI to feature:** cache zależności/buildów, równoległość, uruchamianie tylko **affected** ([monorepo: Turborepo/Nx](../08-architektura-aplikacji/03-microfrontends-i-monorepo.md)). Wolny pipeline = rzadsze integracje i obchodzenie.

## Preview deployments — killer feature frontendu

Każdy PR dostaje **własny, działający URL** (Vercel/Netlify/Cloudflare Pages robią to automatycznie; [hosting](./02-hosting-i-deployment.md)):

- Recenzent/PO/designer klika i **widzi zmianę** zamiast czytać diff — [code review](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md) UI staje się realny.
- E2E/visual regression/Lighthouse odpalają się na preview (środowisko ≈ produkcja).
- Znika „u mnie działa" — wspólny, powtarzalny URL.

## Strategie wdrożeń (deployment)

Ograniczanie ryzyka wypuszczenia zepsutej wersji:

- **Blue-Green** — dwa identyczne środowiska; przełącz ruch na nowe, stare trzymaj jako natychmiastowy rollback.
- **Canary** — nowa wersja dla małego % użytkowników; monitoruj [metryki/błędy](./03-monitoring-i-obserwabilnosc.md), stopniowo zwiększaj lub cofnij.
- **Rolling** — stopniowa wymiana instancji.
- **Feature flags** — deploy ≠ release: kod na produkcji, ale funkcja włączana zdalnie/stopniowo/dla segmentu ([praca z legacy](../15-inzynieria-oprogramowania/03-praca-z-legacy.md)). Frontendowo najpotężniejsze — rollback bez redeploya.

**Rollback musi być trywialny i szybki** — najważniejsza właściwość dojrzałego wdrożenia. Dla statyk frontendu: powrót do poprzedniego, wersjonowanego artefaktu ([cache busting](../09-wydajnosc/04-strategie-cachowania.md) sprawia, że stare i nowe pliki współistnieją).

## Zasady i pułapki specyficzne dla frontendu

- **Cache po deployu:** HTML `no-cache`, statyki z hashem `immutable`; niezgodność = użytkownicy z nowym HTML proszą o stare chunki → błędy ([cache](../09-wydajnosc/04-strategie-cachowania.md)).
- **Zmienne środowiskowe wbudowywane w build** — inny build per środowisko; sekrety **nie** w bundlu frontendu ([bezpieczeństwo](../11-bezpieczenstwo/01-ataki-na-frontend.md)).
- **Migracje długo żyjących sesji** — użytkownik z otwartą kartą na starej wersji po deployu (obsłuż „dostępna nowa wersja, odśwież"; [Service Worker](../08-architektura-aplikacji/02-spa-mpa-pwa.md)).
- **Deterministyczne buildy** — `npm ci` + lockfile ([pakiety](../14-narzedzia/02-pakiety-i-zaleznosci.md)).

## Pułapki i częste błędy

- Brak `tsc --noEmit` w CI — typy „sprawdzane" tylko w edytorze, błędy przechodzą (bundler transpiluje bez sprawdzania).
- `npm install` zamiast `npm ci` — niedeterministyczne buildy.
- Wolny pipeline (bez cache/równoległości) → ludzie obchodzą i integrują rzadziej.
- Brak strategii rollbacku — panika przy zepsutym deployu.
- E2E blokujące każdy commit (wolne, flaky) zamiast rozsądnego triggera.
- Sekrety w logach CI / w bundlu.
- Deploy bez [monitoringu](./03-monitoring-i-obserwabilnosc.md) — nie wiesz, że wersja jest zepsuta, aż zgłoszą użytkownicy.

## Pytania rekrutacyjne

1. **CI vs CD vs Continuous Deployment?** — integracja/gotowość/automatyczny deploy; warunki każdego.
2. **Co składa się na pipeline frontendowy?** — lint, typecheck, testy, build, bramki jakości; kolejność fail-fast.
3. **Do czego preview deployments?** — realne review UI, testy na środowisku ≈ prod.
4. **Canary vs blue-green vs feature flags?** — ograniczanie ryzyka; deploy≠release; rollback.
5. **Czemu `npm ci` i `tsc --noEmit` w CI?** — determinizm; typy nie sprawdzane przez bundler.
6. **Jak zapewniasz szybki rollback frontendu?** — wersjonowane artefakty, feature flags, poprzedni build.

## Dalsza lektura

- [GitHub Actions — dokumentacja](https://docs.github.com/en/actions)
- [martinfowler.com: ContinuousIntegration / FeatureFlags](https://martinfowler.com/articles/continuousIntegration.html)
- [web.dev: Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci)
