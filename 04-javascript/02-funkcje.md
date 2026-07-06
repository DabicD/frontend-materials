# Funkcje

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Typy i zmienne](./01-typy-i-zmienne.md)

W JavaScript funkcje są **obywatelami pierwszej kategorii (first-class)**: są wartościami — można je przypisywać, przekazywać jako argumenty, zwracać z innych funkcji i nadawać im właściwości. Na tym fakcie stoi cały język: callbacki, [closures](./03-scope-hoisting-closures.md), [Promises](./08-asynchronicznosc.md), komponenty, hooki.

## Trzy sposoby definiowania

```js
// 1. Deklaracja (function declaration) — hoistowana W CAŁOŚCI
function add(a, b) { return a + b; }

// 2. Wyrażenie (function expression) — hoisting jak zmiennej
const add = function (a, b) { return a + b; };

// 3. Arrow function
const add = (a, b) => a + b;
```

Różnice, które mają znaczenie:

| | Deklaracja | Wyrażenie | Arrow |
|---|---|---|---|
| Hoisting | całej funkcji (wywołasz przed definicją) | wg zmiennej (TDZ/undefined) | wg zmiennej |
| Własne `this` | tak | tak | **nie — leksykalne** |
| `arguments` | tak | tak | nie (użyj rest `...args`) |
| Konstruktor (`new`) | tak | tak | nie |
| Skrót dla jednego wyrażenia | — | — | tak (implicit return) |

Arrow functions nie mają własnego `this` — biorą je z otoczenia, w którym powstały. To ich główna „supermoc" i główna pułapka (metody obiektów!) — pełne omówienie: [this i kontekst](./04-this-i-kontekst.md).

Implicit return obiektu wymaga nawiasów: `() => ({ id: 1 })` — bez nich `{}` to blok kodu.

## Parametry

```js
function fn(a, b = 10, ...rest) { }   // domyślne + rest (prawdziwa tablica)
fn(1, undefined, 3, 4);               // b=10 (undefined aktywuje domyślną; null NIE)

function draw({ x = 0, y = 0 } = {}) { }   // destrukturyzacja + domyślny obiekt
```

- **Rest** `...rest` zbiera nadmiarowe argumenty; zamiennik archaicznego `arguments` (który nie jest tablicą).
- Parametry domyślne liczone **przy każdym wywołaniu** (w przeciwieństwie do np. Pythona).
- Argumenty-obiekty przekazywane są **przez wartość referencji**: funkcja może zmutować obiekt wywołującego, ale przypisanie nowej wartości do parametru nie zmienia zmiennej na zewnątrz.

## Funkcje wyższego rzędu (higher-order functions)

Funkcja przyjmująca lub zwracająca funkcję. Codzienność JS:

```js
[1, 2, 3].map(x => x * 2);                       // callback
const withLog = fn => (...args) => {              // dekorator/wrapper
  console.log('call', args);
  return fn(...args);
};
const multiply = a => b => a * b;                 // currying
const double = multiply(2);
```

Na tym wzorcu stoją: `map/filter/reduce` ([kolekcje](./06-kolekcje-i-iteracja.md)), debounce/throttle ([wydajność](../09-wydajnosc/03-optymalizacja-dzialania.md)), middleware, HOC-e w frameworkach.

## IIFE i bloki

```js
(function () { /* prywatny scope */ })();
(() => { … })();
```

IIFE (Immediately Invoked Function Expression) — historyczny sposób izolacji zmiennych przed erą [modułów](./10-moduly.md) i bloków `let`. Dziś głównie: inicjalizacja async na najwyższym poziomie starych środowisk, legacy. Warto rozpoznawać.

## Czyste funkcje i efekty uboczne

**Pure function:** wynik zależy tylko od argumentów + brak efektów ubocznych (mutacji, I/O, losowości). Zalety: testowalność, przewidywalność, memoizacja. Frameworki opierają się o czystość renderowania ([model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)), a Redux o czyste reducery. Nie wszystko może być czyste — chodzi o **wypychanie efektów na brzegi** programu.

```js
const addItem = (arr, item) => [...arr, item];   // czysta: nowa tablica
const addItemBad = (arr, item) => { arr.push(item); return arr; };  // mutacja wejścia
```

## Pułapki i częste błędy

- Arrow function jako metoda obiektu / handler wymagający `this` elementu — `this` nie będzie tym, czego oczekujesz.
- `return` i nowa linia: `return\n{ x: 1 }` zwraca `undefined` (ASI wstawia średnik).
- Wywołanie z nadzieją na hoisting wyrażenia funkcyjnego (`const`) — TDZ/ReferenceError.
- Mutowanie argumentów-obiektów — zaskakujące działanie na odległość.
- `null` nie aktywuje parametru domyślnego (tylko `undefined`).

## Pytania rekrutacyjne

1. **Co znaczy, że funkcje są first-class citizens?** — są wartościami: przekazywanie, zwracanie, przypisywanie; konsekwencje (callbacki, HOF).
2. **Deklaracja vs wyrażenie funkcji — różnice?** — hoisting całości vs zmiennej; kiedy to widać.
3. **Czym arrow function różni się od zwykłej?** — this leksykalne, brak arguments/new/prototype; kiedy NIE używać.
4. **Co to funkcja czysta i po co dążyć do czystości?** — determinizm + brak side effects; testy, memoizacja, przewidywalność.
5. **Napisz funkcję `once(fn)` wykonującą `fn` tylko raz.** — klasyka łącząca HOF z [closures](./03-scope-hoisting-closures.md).

## Dalsza lektura

- [MDN: Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
- [javascript.info: Advanced working with functions](https://javascript.info/advanced-functions)
