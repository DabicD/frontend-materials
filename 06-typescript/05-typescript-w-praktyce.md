# TypeScript w praktyce: tsconfig, walidacja, migracja

> **Poziom:** 🟡 średni → 🔴 zaawansowany (decyzje projektowe)
> **Wymagana wiedza:** [Podstawy typowania](./01-podstawy-typowania.md), [Moduły](../04-javascript/10-moduly.md)

Znajomość składni TS to połowa pracy; druga połowa to **operowanie nim w projekcie**: konfiguracja, typy dla bibliotek, walidacja danych na granicach i strategia wprowadzania TS do legacy.

## tsconfig.json — flagi, które musisz rozumieć

```jsonc
{
  "compilerOptions": {
    "strict": true,                        // pakiet flag — fundament (patrz podstawy)
    "target": "ES2022",                    // do jakiej składni JS kompilować
    "module": "ESNext",                    // format modułów w outpucie
    "moduleResolution": "bundler",         // jak szukać modułów (bundler = nowoczesny frontend)
    "lib": ["ES2022", "DOM", "DOM.Iterable"],   // jakie API "istnieją" (typy globalne)
    "noEmit": true,                        // tylko sprawdzaj typy (JS robi bundler)
    "skipLibCheck": true,                  // nie sprawdzaj node_modules (czas kompilacji)
    "noUncheckedIndexedAccess": true,      // arr[i] → T | undefined (warto!)
    "verbatimModuleSyntax": true,          // jawne `import type` (czyste granice typów)
    "paths": { "@/*": ["./src/*"] }        // aliasy (musi znać też bundler!)
  }
}
```

**Model mentalny nowoczesnego frontendu:** TS **nie kompiluje** Twojego kodu — robi to bundler/esbuild/SWC (szybka transpilacja **bez** sprawdzania typów; [bundlery](../14-narzedzia/03-bundlery-i-proces-budowania.md)). TS działa obok: edytor (na żywo) + `tsc --noEmit` w [CI](../16-devops-dla-frontendu/01-ci-cd.md) jako bramka. Dlatego błędy typów **nie blokują** dev-servera — i dlatego CI musi je łapać.

## Deklaracje typów (`.d.ts`)

- Biblioteki niosą typy same (`"types"` w package.json) albo przez pakiety **`@types/*`** (DefinitelyTyped) — np. `@types/node`.
- Własne deklaracje — gdy biblioteka nie ma typów lub dla zasobów niestandardowych:

```ts
// global.d.ts
declare module '*.svg' { const url: string; export default url; }
declare module 'legacy-lib' { export function init(opts: object): void; }
interface Window { dataLayer: unknown[]; }   // rozszerzenie globali (merging)
```

- `declare` = „to istnieje w runtime, wierz mi" — deklaracja, nie implementacja.

## Walidacja na granicach — TS to za mało

Typy znikają w runtime ([type erasure](./01-podstawy-typowania.md)); odpowiedź API może łamać kontrakt. Standard: **schema walidująca + typ wyprowadzony z niej** (jedno źródło prawdy):

```ts
import { z } from 'zod';

const User = z.object({
  id: z.number(),
  name: z.string(),
  role: z.enum(['admin', 'viewer']),
});
type User = z.infer<typeof User>;                 // typ ZA DARMO ze schemy

const data = User.array().parse(await res.json()); // runtime check + typowane dane
```

- Alternatywy: Valibot, ArkType; podejście „standard schema" ujednolica ekosystem.
- Waliduj **granice**: odpowiedzi API, localStorage, query params, dane z postMessage. Wnętrze aplikacji może ufać typom.
- Przy kontraktach API lepszy jeszcze **codegen**: OpenAPI → typy klienta, GraphQL codegen ([GraphQL](../12-komunikacja-z-api/02-graphql.md)) — typy zawsze zsynchronizowane z backendem.

## Migracja JS → TS (strategia)

1. **Włącz TS obok JS:** `allowJs: true`, nowe pliki w TS, stare działają.
2. **Zacznij od granic:** typy domenowe (`User`, `Order`), warstwa API, utils — największy zwrot.
3. **Strict od dnia 1 dla nowych plików**; dla starych — flagi stopniowo lub narzędzia typu `ts-migrate`/`typescript-strict-plugin` (strict per plik).
4. `// @ts-expect-error` (z opisem!) zamiast `@ts-ignore` — pierwszy **protestuje, gdy błąd zniknie** (samosprzątające się długi).
5. Mierz postęp (liczba `any`/ignore w CI) i egzekwuj kierunek: liczba może tylko spadać.

Refaktoryzacyjne podejście krok-po-kroku: [praca z legacy](../15-inzynieria-oprogramowania/03-praca-z-legacy.md).

## Ergonomia i wydajność

- **TS jako serwer językowy** to też autouzupełnianie/refaktory w edytorze — typy są dokumentacją na żywo (hover = kontrakt).
- Wolny `tsc` w monorepo: **project references**, `skipLibCheck`, mniej „sprytnych" typów; przepisany na Go `tsc` (TS 7 / natywny port) drastycznie przyspiesza — śledź stan narzędzi.
- `import type { User } from './types'` — nie wciąga modułu do bundla (z `verbatimModuleSyntax` wymóg jawności).

## Pułapki i częste błędy

- Zielony dev-server ≠ poprawne typy — brak `tsc --noEmit` w CI.
- Zaufanie typom na granicach (API „na pewno zwraca User") — waliduj.
- `@ts-ignore` bez opisu i bez daty ważności.
- tsconfig skopiowany z internetu bez zrozumienia `lib`/`target` — np. brak typów DOM albo target ES5 w 2026.
- Aliasy `paths` bez odpowiednika w bundlerze/vitest — „działa w edytorze, wybucha w buildzie".
- Migracja od najtrudniejszych plików zamiast od granic.

## Pytania rekrutacyjne

1. **Jak TS działa w nowoczesnym pipeline (kto kompiluje, kto sprawdza)?** — bundler transpiluje, tsc waliduje; konsekwencje dla CI.
2. **Jak zapewniasz zgodność typów z rzeczywistym API?** — walidacja schemą + infer / codegen z kontraktu.
3. **Plan migracji dużego projektu JS na TS?** — allowJs, granice najpierw, strict progresywnie, metryki.
4. **`@ts-ignore` vs `@ts-expect-error`?** — drugi wygasa razem z błędem.
5. **Które flagi tsconfig uważasz za obowiązkowe i czemu?** — strict, noUncheckedIndexedAccess, skipLibCheck (pragmatyzm), noEmit w setupie z bundlerem.

## Dalsza lektura

- [tsconfig reference](https://www.typescriptlang.org/tsconfig/) + [Total TypeScript: tsconfig cheat sheet](https://www.totaltypescript.com/tsconfig-cheat-sheet)
- [Zod](https://zod.dev/)
- [TypeScript Performance (wiki)](https://github.com/microsoft/TypeScript/wiki/Performance)
