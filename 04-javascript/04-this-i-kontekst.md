# `this` i kontekst wykonania

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Funkcje](./02-funkcje.md), [Scope i closures](./03-scope-hoisting-closures.md)

`this` to najbardziej mitologizowany mechanizm JS. Reguła jest jedna: **`this` zależy od tego, JAK funkcja została wywołana** (nie gdzie zdefiniowana — to odwrotność zasięgu leksykalnego). Wyjątkiem są arrow functions, które `this` w ogóle nie mają.

## Cztery reguły wiązania `this` (w kolejności priorytetu)

**1. `new`** — konstruktor: `this` = świeżo tworzony obiekt ([prototypy](./05-obiekty-i-prototypy.md)).

```js
function User(name) { this.name = name; }
const u = new User('Ala');   // this → nowy obiekt
```

**2. Jawne wiązanie — `call` / `apply` / `bind`:**

```js
fn.call(ctx, a, b);      // wywołaj z this = ctx, argumenty po przecinku
fn.apply(ctx, [a, b]);   // jak call, argumenty tablicą
const bound = fn.bind(ctx, a);   // NOWA funkcja z przypiętym this (+ partial application)
```

`bind` wiąże **trwale** — późniejszy `call` na związanej funkcji nie zmieni `this`. Dlatego raz zbindowany handler można bezpiecznie przekazywać.

**3. Niejawne (implicit) — wywołanie przez kropkę:** `this` = obiekt przed kropką.

```js
const user = {
  name: 'Ala',
  hello() { console.log(this.name); },
};
user.hello();            // 'Ala' — this = user
```

**4. Domyślne — „goła" funkcja:** `this` = `undefined` w strict mode (moduły ESM i klasy są strict!), `window`/`globalThis` w sloppy mode.

## Utrata kontekstu — bug numer jeden

Kropka wiąże `this` **tylko w momencie wywołania**. Oderwanie metody od obiektu gubi kontekst:

```js
const hello = user.hello;
hello();                             // undefined — reguła domyślna

setTimeout(user.hello, 100);         // ta sama pułapka — callback woła "goło"
button.addEventListener('click', obj.handle);   // this = button, nie obj!
```

Naprawy:

```js
setTimeout(() => user.hello(), 100);        // wrapper — wywołanie przez kropkę
setTimeout(user.hello.bind(user), 100);     // bind
class Comp { handle = () => { this... } }   // class field z arrow — this instancji
```

## Arrow functions — brak własnego `this`

Arrow **nie ma** `this` (ani `arguments`, ani `new`): używa `this` z **leksykalnego otoczenia** — tak jak zwykłej zmiennej z closure.

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(() => this.seconds++, 1000);   // this z metody start → timer ✔
  },
};
```

**Kiedy arrow szkodzi:**

```js
const obj = {
  name: 'X',
  hello: () => console.log(this.name),   // this z modułu/global — NIE obj ✘
};
el.addEventListener('click', () => this…);  // brak this elementu (bywa zamierzone)
```

Zasada praktyczna: **metody obiektów/klas — skrócona składnia `hello() {}`; callbacki wewnątrz metod — arrow.**

## `this` w innych kontekstach

- **Klasy:** metody wywołane po kropce — instancja; oderwane — `undefined` (strict). Stąd wzorzec class fields z arrow dla handlerów.
- **Handlery DOM:** zwykła funkcja → `this` = element (`event.currentTarget`); arrow → leksykalne. Czytelniej używać `event.currentTarget` ([zdarzenia](../05-dom-i-web-api/02-zdarzenia.md)).
- **Poziom modułu ESM:** `this` = `undefined`.
- `globalThis` — uniwersalny dostęp do obiektu globalnego (przeglądarka/worker/Node).

## Pułapki i częste błędy

- Przekazanie metody jako callback bez bind/wrappera (setTimeout, addEventListener, `array.map(obj.fn)`).
- Arrow jako metoda obiektu lub w `Object.defineProperty`/getterach.
- Podwójny bind: `fn.bind(a).bind(b)` — wygrywa pierwszy.
- Zapominanie, że destrukturyzacja metody (`const { hello } = user`) też odrywa kontekst.
- Poleganie na `this` tam, gdzie prostszy jest jawny argument albo closure — czasem najlepszym rozwiązaniem `this` jest brak `this`.

## Pytania rekrutacyjne

1. **Od czego zależy wartość `this`?** — od sposobu wywołania; 4 reguły + arrow jako wyjątek leksykalny.
2. **Co wypisze:**
   ```js
   const obj = { x: 1, get() { return this.x; } };
   const g = obj.get;
   console.log(obj.get(), g());
   ```
   — `1` i `TypeError/undefined` (strict: this=undefined → czytanie `.x` z undefined rzuca).
3. **Różnice między `call`, `apply`, `bind`?** — natychmiastowe wywołanie (argumenty osobno/tablicą) vs nowa funkcja związana.
4. **Dlaczego arrow function nie nadaje się na metodę obiektu?** — brak własnego this; bierze z modułu/otoczenia.
5. **Jak działa `this` w klasowym handlerze zdarzeń i jak go poprawnie związać?** — oderwanie gubi; bind w konstruktorze albo class field arrow.

## Dalsza lektura

- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [javascript.info: Object methods, "this"](https://javascript.info/object-methods)
- „You Don't Know JS Yet: Objects & Classes" (K. Simpson)
