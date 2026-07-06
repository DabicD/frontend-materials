# Pakiety i zależności (npm/yarn/pnpm)

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Moduły](../04-javascript/10-moduly.md)

Menedżer pakietów zarządza zależnościami: instaluje, wersjonuje, rozwiązuje konflikty. Ekosystem npm to największe repozytorium kodu na świecie — i największa powierzchnia [ataku supply chain](../11-bezpieczenstwo/01-ataki-na-frontend.md). Senior rozumie, co dokładnie dzieje się przy `install` i czemu lockfile jest święty.

## package.json — kontrakt projektu

```jsonc
{
  "name": "app", "version": "1.2.3",
  "type": "module",                         // ESM domyślnie ([moduły])
  "scripts": { "dev": "vite", "build": "vite build", "test": "vitest" },
  "dependencies": { "react": "^18.3.0" },   // potrzebne w runtime produkcji
  "devDependencies": { "vitest": "^2.0.0" },// tylko development/build
  "peerDependencies": { "react": ">=18" },  // "host musi dostarczyć" (biblioteki)
  "engines": { "node": ">=20" },
  "overrides": { "semver": "7.5.4" }        // wymuszenie wersji zależności zależności
}
```

- **dependencies vs devDependencies** — testy/lintery/bundlery to dev (nie trafiają do zależności konsumentów Twojej biblioteki); w aplikacji rozróżnienie mniej krytyczne (i tak budujesz bundle), ale utrzymuj porządek.
- **peerDependencies** — pluginy/biblioteki deklarują „potrzebuję React, ale dostarcza go aplikacja" (uniknięcie dwóch kopii Reacta).
- **scripts** — wejście do zadań projektu; `pre`/`post` hooki; uwaga na `postinstall` (wektor [supply chain](../11-bezpieczenstwo/01-ataki-na-frontend.md)).

## SemVer — wersjonowanie semantyczne

`MAJOR.MINOR.PATCH` (`2.4.1`):
- **PATCH** — poprawki bez zmiany API (bugfix).
- **MINOR** — nowe funkcje, wstecznie zgodne.
- **MAJOR** — zmiany łamiące.

Zakresy w package.json:
- `^1.2.3` — kompatybilne: `>=1.2.3 <2.0.0` (MINOR/PATCH; domyślne w npm).
- `~1.2.3` — tylko PATCH: `>=1.2.3 <1.3.0`.
- `1.2.3` — dokładnie ta wersja; `*`/`latest` — cokolwiek (niebezpieczne).

SemVer to **obietnica**, nie gwarancja — MINOR bywa łamiący (bug). Stąd lockfile.

## Lockfile — dlaczego jest święty

`package.json` deklaruje **zakresy**; lockfile (`package-lock.json`/`yarn.lock`/`pnpm-lock.yaml`) zapisuje **dokładne wersje całego drzewa** (wraz z zależnościami zależności) + hashe integralności.

- **Commituj lockfile** — gwarantuje, że każdy developer i [CI](../16-devops-dla-frontendu/01-ci-cd.md) instalują **identyczne** wersje. Bez niego „u mnie działa" wynika z innych wersji transitive deps.
- **W CI używaj `npm ci`** (nie `npm install`) — instaluje **dokładnie** z lockfile, wywala się przy niezgodności z package.json, czyści `node_modules`. Deterministyczne buildy.
- Konflikt w lockfile przy merge — nie edytuj ręcznie; regeneruj (`npm install` po rozwiązaniu package.json).

## npm vs yarn vs pnpm

| | npm | yarn (berry) | pnpm |
|---|---|---|---|
| Status | domyślny, wbudowany | dojrzały, PnP | **rosnący faworyt** |
| node_modules | płaski (hoisting) | płaski / PnP | **symlinki + globalny store** |
| Miejsce na dysku | duplikaty między projektami | jw. | **współdzielony store (oszczędność)** |
| Ścisłość | luźna (phantom deps) | — | **ścisła (tylko zadeklarowane widoczne)** |

- **pnpm** — jedna kopia każdej wersji w globalnym store, do projektów linkowana; szybki, oszczędny, **rygorystyczny** (blokuje „phantom dependencies" — używanie paczki, której nie zadeklarowałeś, a była dostępna przez hoisting). Domyślny wybór wielu nowych projektów, zwłaszcza [monorepo](../08-architektura-aplikacji/03-microfrontends-i-monorepo.md) (workspaces).
- **Nie mieszaj** menedżerów w jednym repo (różne lockfile → chaos).

## Bezpieczeństwo i higiena zależności

Największe realne ryzyko frontendu = [supply chain](../11-bezpieczenstwo/01-ataki-na-frontend.md):

- `npm audit` — znane podatności (dużo szumu/false-positives przy devDeps; czytaj krytycznie).
- **Dependabot/Renovate** — automatyczne PR-y z aktualizacjami; regularne małe update'y > wielki skok raz na rok.
- **Minimalizm** — każda zależność to kod z pełnymi prawami w Twoim [bundle'u](./03-bundlery-i-proces-budowania.md) i wagą ([optymalizacja](../09-wydajnosc/02-optymalizacja-ladowania.md)); zanim dodasz paczkę do trywialnej rzeczy (left-pad!), rozważ własne kilka linii lub API natywne.
- **Weryfikuj** przed instalacją: aktywność, maintainerzy, liczba i waga zależności (bundlephobia), typosquatting (literówki w nazwie), `postinstall`.
- **Lockfile + `npm ci`** to też bezpieczeństwo — hashe integralności wykrywają podmianę.
- **`npx`** uruchamia paczkę bez instalacji — wygodne, ale to też wykonanie cudzego kodu.

## Publikowanie (świadomość)

Pole `"exports"` (mapy ESM/CJS/typy), `"files"`, `"types"`, `.npmignore`, `npm publish`, wersjonowanie (changesets w monorepo). Rzadziej robione przez frontendowca aplikacyjnego, częste przy bibliotekach/design systemach.

## Pułapki i częste błędy

- `npm install` w CI zamiast `npm ci` — niedeterministyczne buildy.
- Nie commitowanie lockfile albo jego ręczna edycja.
- Mieszanie menedżerów (npm + yarn) w jednym projekcie.
- `latest`/`*`/brak zakresu — losowe zmiany przy instalacji.
- Instalowanie paczek bez oceny (waga, zależności, maintainer) — dług i ryzyko.
- Ignorowanie aktualizacji przez rok → bolesny, ryzykowny wielki upgrade.
- `dependencies` vs `devDependencies` pomieszane w publikowanej bibliotece — konsumenci ciągną Twoje narzędzia dev.

## Pytania rekrutacyjne

1. **Do czego służy lockfile i czemu się go commituje?** — dokładne wersje całego drzewa; deterministyczne, identyczne instalacje.
2. **`npm install` vs `npm ci`?** — zakresy vs dokładnie-z-lockfile; ci dla CI.
3. **Wyjaśnij `^` vs `~` w SemVer.** — MINOR+PATCH vs tylko PATCH.
4. **dependencies vs devDependencies vs peerDependencies?** — runtime / dev / dostarczane przez hosta.
5. **Jak dbasz o bezpieczeństwo zależności?** — audit, Renovate/Dependabot, minimalizm, weryfikacja, lockfile+ci.
6. **Czemu pnpm zyskuje popularność?** — store+symlinki (miejsce/szybkość), ścisłość (brak phantom deps).

## Dalsza lektura

- [npm docs: package.json](https://docs.npmjs.com/cli/configuring-npm/package-json) + [semver](https://semver.org/lang/pl/)
- [pnpm — motivation](https://pnpm.io/motivation)
- [Bundlephobia](https://bundlephobia.com/) — waga paczki przed instalacją
