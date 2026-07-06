# Składnia ES6+ (nowoczesny JavaScript)

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Typy i zmienne](./01-typy-i-zmienne.md), [Funkcje](./02-funkcje.md)

ES6/ES2015 zapoczątkował coroczne wydania języka (ES2016, ES2017…). Ten plik zbiera składnię, którą piszesz dziesiątki razy dziennie — z detalami, o które pytają na rozmowach. (Osobne domy mają: [let/const](./01-typy-i-zmienne.md), [arrow functions](./02-funkcje.md), [klasy](./05-obiekty-i-prototypy.md), [Promises/async](./08-asynchronicznosc.md), [moduły](./10-moduly.md).)

## Destrukturyzacja

```js
// Obiekty: po NAZWACH właściwości
const { name, age = 18, address: { city } = {}, ...rest } = user;
//    zmiana nazwy ─┐
const { name: userName } = user;

// Tablice: po POZYCJI (działa z każdym iterable)
const [first, , third, ...tail] = arr;
const [a, b] = [b, a];                      // swap bez zmiennej tymczasowej

// W parametrach funkcji (z domyślnym obiektem — wywołanie bez argumentu nie wybucha)
function draw({ x = 0, y = 0 } = {}) {}

// W pętli po entries
for (const [key, value] of Object.entries(obj)) {}
```

Pułapka: destrukturyzacja `null`/`undefined` rzuca TypeError — stąd wzorce `= {}` i optional chaining przy źródle danych.

## Spread i rest

Ten sam zapis `...`, dwie role:

```js
// SPREAD — "rozpakuj" (w wywołaniu / literale)
Math.max(...nums);
const copy = [...arr];                  // płytka kopia
const merged = { ...defaults, ...options };   // późniejsze nadpisują wcześniejsze
const updated = { ...user, age: 31 };   // niemutująca "zmiana" — fundament pracy ze stanem

// REST — "zbierz resztę" (w deklaracji)
function log(first, ...args) {}
const { id, ...withoutId } = user;      // "usunięcie" pola bez mutacji
```

Spread obiektu kopiuje **płytko** — [typy: wartość vs referencja](./01-typy-i-zmienne.md).

## Template literals

```js
const msg = `Cześć, ${user.name}! Masz ${items.length} pozycji.`;
const html = `wielo
liniowy`;
const query = sql`SELECT * FROM users WHERE id = ${id}`;   // tagged template
```

Tagged templates (funkcja przed backtickiem dostaje części stringa i wartości) napędzają m.in. styled-components czy biblioteki SQL/i18n — warto rozpoznawać.

## Optional chaining `?.` i nullish coalescing `??`

```js
const city = user?.address?.city;         // undefined zamiast TypeError
user?.getName?.();                        // wywołanie, jeśli funkcja istnieje
arr?.[0];                                  // indeks
```

`?.` skraca obliczenie (short-circuit), gdy lewa strona to `null`/`undefined` — **tylko te dwie wartości**.

```js
const port = config.port ?? 3000;   // 0 zostaje 0!  (?? reaguje tylko na null/undefined)
const port = config.port || 3000;   // 0, "" i false też zamienia — częsty bug
```

Operatory przypisania: `??=`, `||=`, `&&=`. Zasada: **do wartości domyślnych `??`**, `||` tylko gdy świadomie chcesz odrzucić wszystkie falsy.

## Skróty obiektowe i computed properties

```js
const x = 1, y = 2;
const point = { x, y };                        // shorthand
const obj = { [`key_${id}`]: true };           // computed key
const api = { get() {}, set() {} };            // method shorthand
```

## Wybrane nowości ES2020+ (przekrojowo)

- `Promise.allSettled`, `Promise.any` — [asynchroniczność](./08-asynchronicznosc.md).
- `String.prototype.replaceAll`, `at(-1)`, `Object.fromEntries`, `Object.groupBy`.
- `structuredClone()` — głęboka kopia bez hacków `JSON.parse(JSON.stringify(...))` (który gubi `Date`, `Map`, `undefined`, funkcje i wybucha na cyklach).
- Top-level `await` (w modułach ESM), `import.meta`.
- `Array.fromAsync`, immutable metody tablic (`toSorted`… — [kolekcje](./06-kolekcje-i-iteracja.md)).
- `Temporal` — nowe API dat zastępujące upośledzony `Date` (już w silnikach; śledź wsparcie).

Nie musisz znać numerów wydań — musisz wiedzieć, **co jest natywne**, żeby nie dopisywać lodasha do rzeczy wbudowanych. Transpilacja starych targetów: [bundlery i build](../14-narzedzia/03-bundlery-i-proces-budowania.md).

## Pułapki i częste błędy

- `||` zamiast `??` przy wartościach, gdzie `0`/`""`/`false` są poprawne.
- Nadużywanie `?.` — maskowanie błędów danych („czemu wszędzie undefined?") zamiast walidacji przy źródle.
- Głębokie „kopiowanie" spreadem.
- Destrukturyzacja z długimi łańcuchami zagnieżdżeń — nieczytelna i krucha; czasem zwykłe `const city = user.address.city` jest lepsze.
- Mieszanie rest/spread bez zrozumienia strony deklaracji vs użycia.

## Pytania rekrutacyjne

1. **Różnica między `??` a `||`?** — nullish (null/undefined) vs falsy; przykład z `0`.
2. **Jak „usunąć" pole z obiektu bez mutacji?** — destrukturyzacja z rest: `const { pass, ...safe } = user`.
3. **Co robi `?.` i kiedy przerywa łańcuch?** — short-circuit na null/undefined; zwraca undefined.
4. **Dlaczego `JSON.parse(JSON.stringify(x))` to słaba głęboka kopia?** — gubi typy i wybucha na cyklach; `structuredClone`.
5. **Spread vs rest — wyjaśnij na przykładach.** — rozpakowanie w użyciu vs zbieranie w deklaracji.

## Dalsza lektura

- [javascript.info: Destructuring assignment](https://javascript.info/destructuring-assignment)
- [MDN: Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
- [TC39 finished proposals](https://github.com/tc39/proposals/blob/main/finished-proposals.md) — co weszło do języka i kiedy
