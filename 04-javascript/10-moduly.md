# Moduły: ESM i CommonJS

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Scope i closures](./03-scope-hoisting-closures.md)

Moduł = plik z własnym scope, jawnie eksportujący i importujący. Standard języka to **ESM (ECMAScript Modules)**; **CommonJS (CJS)** to starszy format Node.js, wciąż wszechobecny w ekosystemie npm. Rozumienie różnic ratuje przy błędach typu „cannot use import outside a module" i przy konfiguracji [bundlerów](../14-narzedzia/03-bundlery-i-proces-budowania.md).

## ESM — składnia

```js
// eksporty nazwane (wiele na plik)
export const API_URL = '…';
export function getUser() {}
export { helper as publicHelper };

// eksport domyślny (max 1)
export default class UserService {}

// importy
import UserService from './user-service.js';        // default
import { API_URL, getUser } from './api.js';         // nazwane
import { getUser as fetchUser } from './api.js';     // alias
import * as api from './api.js';                      // namespace
import './setup.js';                                  // sam efekt uboczny
export * from './utils.js';                           // re-eksport (pliki-barrel)
```

**Fakty, które trzeba znać:**
- Importy/eksporty są **statyczne** — na najwyższym poziomie, nazwy znane bez uruchamiania kodu. To umożliwia [tree shaking](../14-narzedzia/03-bundlery-i-proces-budowania.md) i weryfikację nazw w buildzie.
- Import to **żywe wiązanie (live binding)** do eksportowanej zmiennej — nie kopia wartości.
- Importy są **hoistowane**; moduł wykonuje się **raz** (singleton z natury — cache po URL/ścieżce).
- Kod modułu działa w **strict mode**, ma własny scope (nic nie wycieka do global), top-level `await` dozwolony.
- W przeglądarce: `<script type="module">` (defer z natury — [struktura dokumentu](../02-html/01-struktura-dokumentu.md)).
- Grafy cykliczne działają, ale częściowo zainicjalizowane moduły to źródło „undefined is not a function" — unikaj cykli (sygnał złego podziału).

## Dynamiczny import

```js
const { Chart } = await import('./chart.js');   // Promise; ładuje NA ŻĄDANIE
```

Podstawa **code splittingu i lazy loadingu**: bundler wydziela osobny chunk, ładowany dopiero przy wywołaniu ([optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md), [routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)).

## CommonJS — rozpoznawanie i różnice

```js
// eksport
module.exports = { getUser };
exports.helper = () => {};
// import
const { getUser } = require('./api');
```

| | ESM | CJS |
|---|---|---|
| Ładowanie | asynchroniczne możliwe, statyczna analiza | synchroniczne, dynamiczne (`require` gdziekolwiek) |
| Wiązania | live bindings | kopia wartości z momentu require |
| Tree shaking | tak | praktycznie nie |
| Środowisko | standard: przeglądarki + Node | Node (i stare bundle) |
| `this` na top-level | `undefined` | `module.exports` |

W Node: `.mjs`/`.cjs` lub pole `"type": "module"` w package.json; interop (import CJS z ESM działa, odwrotnie — ograniczony przez `createRequire`/dynamiczny import). Pakiety deklarują oba warianty przez pole `"exports"` ([pakiety i zależności](../14-narzedzia/02-pakiety-i-zaleznosci.md)).

## Praktyki projektowe

- **Nazwane eksporty > default** w kodzie zespołowym: wymuszona spójność nazw, lepsze auto-importy i refaktory (default pozwala każdemu importować pod inną nazwą).
- **Pliki-barrel (`index.js` z `export *`)** — wygodne, ale: utrudniają tree shaking, potrafią zaciągać pół aplikacji jednym importem i tworzyć cykle. W dużych projektach — ostrożnie/wcale.
- Moduł jako singleton: eksport instancji (`export const store = createStore()`) to najprostszy globalny stan — świadomie, bo utrudnia testowanie ([testy jednostkowe](../13-testowanie/02-testy-jednostkowe.md)).
- Granice modułów i publiczne API katalogów: [struktura projektu](../08-architektura-aplikacji/04-struktura-projektu.md).

## Przed modułami (rozpoznawanie w legacy)

IIFE + obiekt globalny (`window.MyApp = …`), AMD/RequireJS (`define([...], fn)`), UMD (wrapper wykrywający środowisko — tak dystrybuowano biblioteki). Spotykasz je w starych codebase'ach i starych paczkach npm.

## Pułapki i częste błędy

- „SyntaxError: Cannot use import statement outside a module" — plik traktowany jako skrypt/CJS: brak `type="module"` / `"type": "module"` / złe rozszerzenie.
- Miks `require` i `import` w jednym pliku.
- Cykliczne importy przez pliki-barrel — objaw: undefined przy imporcie, zależny od kolejności.
- Efekty uboczne w modułach (kod wykonywany przy imporcie) — zaskakująca kolejność inicjalizacji; pole `"sideEffects"` w package.json wpływa na tree shaking.
- Zapominanie, że moduł wykonuje się raz — mutowanie eksportowanego obiektu widoczne wszędzie.

## Pytania rekrutacyjne

1. **ESM vs CommonJS — kluczowe różnice?** — statyczność, live bindings, tree shaking, async.
2. **Dlaczego statyczna natura importów jest ważna?** — analiza grafu w buildzie: tree shaking, splitting, weryfikacja.
3. **Do czego służy dynamiczny `import()`?** — lazy loading/code splitting; zwraca Promise.
4. **Default vs named exports — Twoja praktyka?** — argumenty za named; kiedy default (konwencje frameworków).
5. **Co to jest plik-barrel i jakie ma wady?** — re-eksporty; tree shaking, cykle, ukryte zależności.

## Dalsza lektura

- [MDN: JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [javascript.info: Modules](https://javascript.info/modules)
- [Node.js docs: ESM ↔ CJS interoperability](https://nodejs.org/api/esm.html)
