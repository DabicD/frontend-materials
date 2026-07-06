# Typy i zmienne

> **Poziom:** 🟢 podstawowy (fundament wszystkiego)
> **Wymagana wiedza:** brak — start sekcji JavaScript

JavaScript jest językiem **dynamicznie i słabo typowanym**: typ ma wartość (nie zmienna), a język sam konwertuje typy, gdy „musi" (coercion). Większość „dziwactw" JS to konsekwencje tych dwóch faktów — zrozumiane raz, przestają być magią.

## Typy danych

**7 typów prymitywnych:** `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`
**+ 1 typ złożony:** `object` (w tym tablice, funkcje, daty, regexy…).

Kluczowa różnica:

- **Prymitywy — kopiowane przez wartość**, niemutowalne (`"abc".toUpperCase()` zwraca nowy string).
- **Obiekty — przez referencję**: zmienna trzyma „adres", przypisanie kopiuje adres, nie obiekt.

```js
let a = { x: 1 };
let b = a;          // ta sama referencja
b.x = 2;
a.x;                // 2 (!) — jeden obiekt, dwie zmienne

let c = { x: 1 };
a === c;            // false — porównanie referencji, nie zawartości
```

Stąd też: płytka vs głęboka kopia (`{ ...obj }` / `structuredClone(obj)`) i czemu w [frameworkach reaktywnych](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md) mutacja obiektu bywa „niewidoczna".

**Detale, które trzeba znać:**
- `typeof null === "object"` — historyczny bug, został na zawsze. Sprawdzanie nulla: `x === null`.
- `undefined` = „nie przypisano"; `null` = „świadomie puste". `x == null` łapie oba (jedyne powszechnie akceptowane użycie `==`).
- `number` to IEEE 754 double: stąd `0.1 + 0.2 !== 0.3` (błąd reprezentacji binarnej — porównuj z epsilonem albo licz w groszach/`bigint`), `NaN !== NaN` (sprawdzaj `Number.isNaN`), bezpieczne całkowite tylko do `Number.MAX_SAFE_INTEGER` (2^53−1) — powyżej `bigint`.
- `symbol` — unikalne klucze właściwości ([zaawansowane](./12-zaawansowane.md)).

## Coercion (konwersja typów)

Jawna: `Number("42")`, `String(42)`, `Boolean(x)`. Niejawna — reguły:

- **Do stringa:** operator `+`, gdy którykolwiek operand jest stringiem: `1 + "2" === "12"`.
- **Do liczby:** pozostałe operatory arytmetyczne: `"6" * "2" === 12`; unarny plus `+x`.
- **Do booleana:** konteksty logiczne (`if`, `!`, `&&`, `||`). **Falsy jest dokładnie 8 wartości:** `false, 0, -0, 0n, "", null, undefined, NaN`. **Wszystko inne truthy** — w tym `"0"`, `[]`, `{}`!

```js
[] + []      // "" (oba do stringa)
[] + {}      // "[object Object]"
"5" - 1      // 4
"5" + 1      // "51"
```

## `==` vs `===`

- `===` — bez konwersji: typy różne → `false`. **Domyślny wybór, zawsze.**
- `==` — z konwersją wg (nieintuicyjnego) algorytmu: `"" == 0` ✔, `"0" == 0` ✔, `null == undefined` ✔, ale `null == 0` ✘.
- `Object.is(a, b)` — jak `===`, ale `NaN` równe sobie i rozróżnia `±0` (używane wewnętrznie przez React do porównań).

## `var` vs `let` vs `const`

| | `var` | `let` | `const` |
|---|---|---|---|
| Zasięg | funkcyjny | blokowy `{}` | blokowy |
| Redeklaracja | tak | nie | nie |
| Hoisting | tak, inicjalizowany `undefined` | tak, ale **TDZ** | tak, TDZ |
| Ponowne przypisanie | tak | tak | **nie** |
| Na `window` (global) | tak | nie | nie |

- **TDZ (Temporal Dead Zone):** `let`/`const` są hoistowane, ale dostęp przed deklaracją rzuca `ReferenceError` (a nie zwraca `undefined` jak `var`). Szczegóły hoistingu: [scope i closures](./03-scope-hoisting-closures.md).
- **`const` nie mrozi obiektu** — blokuje tylko ponowne przypisanie zmiennej. `const arr = []; arr.push(1)` jest legalne. Zamrożenie (płytkie): `Object.freeze(obj)`.
- **Praktyka:** domyślnie `const`, `let` gdy wartość faktycznie się zmienia, `var` nigdy (spotkasz w legacy).

Klasyka rekrutacyjna — pętla z `var`:

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i));   // 3, 3, 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i));   // 0, 1, 2
```

`var` = jedna wspólna zmienna; `let` = nowa zmienna per iteracja (domknięcie łapie osobne — [closures](./03-scope-hoisting-closures.md), [event loop](./09-event-loop.md)).

## Sprawdzanie typów w praktyce

```js
typeof x            // prymitywy i "function"; uwaga: null → "object"
Array.isArray(x)    // tablica (typeof da "object")
x instanceof Date   // sprawdzenie po łańcuchu prototypów
x === null
Number.isNaN(x), Number.isFinite(x), Number.isInteger(x)
```

## Pułapki i częste błędy

- Porównywanie obiektów/tablic przez `===` z nadzieją na porównanie zawartości.
- Płytki spread `{ ...obj }` traktowany jak głęboka kopia — zagnieżdżone obiekty wciąż współdzielone.
- Arytmetyka na pieniądzach we floatach — grosze jako liczby całkowite albo biblioteka dziesiętna.
- `if (x)` jako test „czy zdefiniowane" — odrzuca też `0` i `""`; precyzyjniej `x !== undefined` / `x != null` / `??` ([ES6+](./07-skladnia-es6-plus.md)).
- Poleganie na `typeof x === "object"` bez wykluczenia `null` i tablic.

## Pytania rekrutacyjne

1. **Wymień typy prymitywne. Czym różnią się od obiektów?** — 7 prymitywów; wartość vs referencja, niemutowalność.
2. **Dlaczego `0.1 + 0.2 !== 0.3`?** — binarna reprezentacja IEEE 754; jak liczyć pieniądze.
3. **Czym jest TDZ?** — okno między hoistingiem a deklaracją `let`/`const`; ReferenceError zamiast cichego `undefined`.
4. **Co wypisze pętla z `var` + setTimeout i czemu?** — patrz wyżej; scope funkcyjny vs blokowy + domknięcia.
5. **Wymień wartości falsy.** — 8 sztuk; podkreśl, że `[]` i `{}` są truthy.
6. **`==` vs `===` vs `Object.is`?** — coercion / brak / NaN i ±0.

## Dalsza lektura

- [MDN: JavaScript data types and data structures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures)
- [javascript.info: Data types](https://javascript.info/types)
- „You Don't Know JS Yet" — tom *Types & Grammar* (K. Simpson)
