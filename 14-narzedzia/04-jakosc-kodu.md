# Jakość kodu: ESLint, Prettier, hooki

> **Poziom:** 🟢 podstawowy → 🟡 (konfiguracja zespołowa)
> **Wymagana wiedza:** [Pakiety i zależności](./02-pakiety-i-zaleznosci.md)

Narzędzia jakości utrzymują kod **spójny i wolny od całej klasy błędów** — automatycznie, zanim trafi do review. Kluczowe rozróżnienie, które wielu myli: **linter szuka błędów (jakość), formatter dba o wygląd (styl)** — to dwa różne narzędzia o rozłącznych zadaniach.

## ESLint — analiza statyczna

Wykrywa **problemy logiczne i antywzorce**, nie tylko styl: nieużywane zmienne, `==` zamiast `===`, brakujące zależności efektów, niebezpieczne wzorce, naruszenia [reguł a11y](../10-dostepnosc/03-a11y-w-praktyce.md) i [granic modułów](../08-architektura-aplikacji/04-struktura-projektu.md).

```js
// eslint.config.js — "flat config" (nowy standard, zastąpił .eslintrc)
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,
  { rules: {
      'no-console': 'warn',
      '@typescript-eslint/no-floating-promises': 'error',   // złapany zgubiony await!
  } },
];
```

- **Pluginy** rozszerzają reguły: `typescript-eslint` (typy — łapie [floating promises](../04-javascript/08-asynchronicznosc.md)), `eslint-plugin-jsx-a11y`, plugin frameworka (reguły hooków), `import`/`boundaries` ([struktura](../08-architektura-aplikacji/04-struktura-projektu.md)).
- **`--fix`** naprawia część automatycznie.
- Poziomy: `off`/`warn`/`error` (error blokuje [CI](../16-devops-dla-frontendu/01-ci-cd.md)).
- Reguły „type-aware" (wymagają typów TS) łapią więcej, ale są wolniejsze — świadomy kompromis.

## Prettier — formatowanie

Automatyczny, opiniowany formatter: łamanie linii, wcięcia, cudzysłowy, średniki, przecinki. **Kończy dyskusje stylistyczne** — nie ma „mojego stylu", jest jeden output.

- Formatuje na zapis (edytor) i w [pre-commit hooku](#hooki).
- **Rozdział odpowiedzialności:** Prettier = wygląd; ESLint = jakość. Historyczne reguły stylistyczne w ESLint są dziś deprecated — nie duplikuj (Prettier formatuje, ESLint się nie wtrąca do wyglądu). Alternatywy łączące oba: **Biome** (Rust, linter+formatter w jednym, bardzo szybki) — rosnąca opcja.

## Hooki Git — brama jakości

**Husky** + **lint-staged** — uruchamiają narzędzia automatycznie przy operacjach Git:

```jsonc
// lint-staged: działaj TYLKO na plikach w commicie (szybko)
"lint-staged": {
  "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
  "*.{css,md}": ["prettier --write"]
}
```

- **pre-commit** — format + lint zmienionych plików (szybko, bo tylko staged).
- **pre-push / commit-msg** — cięższe testy / walidacja formatu wiadomości.
- **commitlint** — egzekwuje [Conventional Commits](#conventional-commits).

**Zasada:** hooki to szybka informacja zwrotna dla developera, **nie jedyna brama** — mogą być pominięte (`--no-verify`), więc [CI](../16-devops-dla-frontendu/01-ci-cd.md) musi powtórzyć te same sprawdzenia. Hook lokalnie = wygoda; CI = gwarancja.

## Conventional Commits

Konwencja wiadomości: `type(scope): opis` — `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`.

```
feat(cart): dodaj usuwanie pozycji
fix(auth): obsłuż wygaśnięcie tokena
feat!: nowy format API   ← "!" = breaking change (MAJOR)
```

Wartość: czytelna historia ([git](./01-git.md)), automatyczny **changelog** i **wersjonowanie** ([SemVer](./02-pakiety-i-zaleznosci.md)) przez semantic-release/changesets, filtrowalne logi. Egzekwowane przez commitlint w hooku.

## Dodatkowe warstwy jakości

- **TypeScript** — najgłębsza analiza statyczna ([TS](../06-typescript/01-podstawy-typowania.md)); `tsc --noEmit` w CI.
- **Stylelint** — linting CSS/SCSS ([preprocesory](../03-css/10-preprocesory-i-narzedzia.md)).
- **EditorConfig** — bazowe ustawienia edytora (wcięcia, końce linii) niezależne od IDE.
- **knip / depcheck** — martwy kod i nieużywane zależności.

## Pułapki i częste błędy

- Duplikowanie stylistycznych reguł w ESLint i Prettier — konflikty, „walczące" narzędzia.
- Hooki jako jedyna brama (bez CI) — pomijalne przez `--no-verify`.
- Ciężki lint całego repo w pre-commit zamiast `lint-staged` — wolne commity, frustracja, obchodzenie.
- `eslint-disable` rozsiane wszędzie zamiast naprawy/dyskusji o regule — martwe reguły.
- Wprowadzanie ESLinta do dużego legacy z tysiącami błędów naraz → zespół wyłącza; wchodź stopniowo (warn → error, per katalog).
- Reguły „bo tak w tutorialu" bez konsensusu zespołu — konfiguracja to decyzja zespołowa ([code review](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md)).

## Pytania rekrutacyjne

1. **Linter vs formatter — różnica?** — jakość/logika vs wygląd; ESLint vs Prettier; nie duplikować.
2. **Jak zapewniasz spójność jakości w zespole?** — ESLint+Prettier+TS, hooki (husky+lint-staged), CI powtarza.
3. **Czemu hooki nie wystarczą?** — pomijalne; CI jako twarda brama.
4. **Co dają Conventional Commits?** — czytelność, automatyczny changelog/wersjonowanie.
5. **Jak wdrożysz lintera do dużego legacy?** — stopniowo: warn→error, per moduł, autofix, baseline.

## Dalsza lektura

- [ESLint — dokumentacja (flat config)](https://eslint.org/docs/latest/)
- [Prettier — Prettier vs. Linters](https://prettier.io/docs/en/comparison.html)
- [Conventional Commits](https://www.conventionalcommits.org/pl/)
- [Biome](https://biomejs.dev/) — szybka alternatywa lint+format
