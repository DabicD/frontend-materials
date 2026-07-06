# Scope, hoisting i closures

> **Poziom:** 🟡 średni (i absolutny hit rozmów rekrutacyjnych)
> **Wymagana wiedza:** [Typy i zmienne](./01-typy-i-zmienne.md), [Funkcje](./02-funkcje.md)

Scope określa, **skąd widać zmienną**; hoisting — **co silnik robi z deklaracjami przed uruchomieniem kodu**; closure — **jak funkcja pamięta zmienne z miejsca narodzin**. Te trzy mechanizmy to jeden spójny system — poznany porządnie, tłumaczy zachowania, które inaczej wyglądają jak magia.

## Scope (zasięg)

- **Globalny** — najwyższy poziom; w przeglądarce `var`/deklaracje funkcji lądują na `window` (moduły ESM mają własny scope!).
- **Funkcyjny** — każda funkcja tworzy scope; widzi własne zmienne + zewnętrzne (scope chain).
- **Blokowy** — `{}` z `let`/`const` (od ES6); `var` go **ignoruje**.

**Zasięg leksykalny (lexical/static scope):** o tym, co funkcja widzi, decyduje **miejsce jej definicji w kodzie**, nie miejsce wywołania. (Kontrast: `this`, które zależy od wywołania — [this i kontekst](./04-this-i-kontekst.md)).

```js
const x = 'globalny';
function outer() {
  const x = 'zewnętrzny';
  function inner() { console.log(x); }   // szuka: inner → outer → global
  return inner;
}
outer()();   // 'zewnętrzny' — nieważne, że wywołana globalnie
```

**Shadowing** — zmienna wewnętrzna przesłania zewnętrzną o tej samej nazwie.

## Hoisting

Przed wykonaniem kodu silnik skanuje scope i **rejestruje deklaracje**:

- **Deklaracje funkcji** — hoistowane z całym ciałem (można wywołać „przed" definicją).
- **`var`** — hoistowany i **inicjalizowany `undefined`** (dostęp przed linią deklaracji: `undefined`, cicho).
- **`let`/`const`/`class`** — hoistowane, ale **nieinicjalizowane**: dostęp przed deklaracją = `ReferenceError` (**Temporal Dead Zone**).

```js
console.log(a);   // undefined  (var — hoisting z undefined)
console.log(b);   // ReferenceError (TDZ)
var a = 1;
let b = 2;
```

Mentalny model: „deklaracje idą na górę scope'u; przypisania zostają na miejscu". TDZ to celowy design — ujawnia błędy, które `var` maskował.

## Closures (domknięcia)

**Closure = funkcja + referencje do zmiennych z leksykalnego otoczenia, w którym powstała.** Funkcja „domyka" zmienne — żyją tak długo, jak funkcja, nawet gdy zewnętrzna funkcja dawno się zakończyła.

```js
function createCounter() {
  let count = 0;                    // prywatna — niedostępna z zewnątrz
  return {
    increment: () => ++count,
    get: () => count,
  };
}
const counter = createCounter();
counter.increment(); counter.increment();
counter.get();        // 2
counter.count;        // undefined — enkapsulacja!
```

**Kluczowe fakty:**
- Domykana jest **zmienna (referencja), nie wartość** — closure widzi aktualny stan, nie snapshot z momentu utworzenia.
- Każde wywołanie funkcji zewnętrznej tworzy **nowe, niezależne** środowisko (dwa `createCounter()` = dwa liczniki).
- Na closures stoją: callbacki i handlery (pamiętają kontekst), [moduły](./10-moduly.md), debounce/throttle, memoizacja, fabryki, „prywatność" przed erą `#pól` klas, hooki Reactowe (i ich słynne „stale closure").

**Klasyka: pętla `var` vs `let`** — wyjaśniona w [typach i zmiennych](./01-typy-i-zmienne.md): z `var` wszystkie callbacki domykają **jedną** zmienną; z `let` każda iteracja ma własną.

**Praktyczne wzorce:**

```js
// once — funkcja jednorazowa
const once = fn => {
  let done = false, result;
  return (...args) => done ? result : (done = true, result = fn(...args));
};

// memoize — cache wyników
const memoize = fn => {
  const cache = new Map();
  return arg => cache.has(arg) ? cache.get(arg) : cache.set(arg, fn(arg)).get(arg);
};
```

**Closures a pamięć:** domknięte zmienne nie są zbierane przez GC, dopóki żyje funkcja — długowieczne closure trzymające duże obiekty (albo węzły DOM) to typowe źródło **memory leaków** ([zaawansowane](./12-zaawansowane.md)).

## Pułapki i częste błędy

- Oczekiwanie, że closure „zamroziło" wartość — widzi bieżący stan zmiennej (stąd „stale closure" to odwrotny problem: stary **snapshot przez nową zmienną** w kolejnych renderach frameworka).
- `var` w pętli + async callback — patrz klasyka.
- Nieświadome trzymanie referencji do wielkich struktur w handlerach — leaki.
- Mylenie zasięgu leksykalnego z dynamicznym `this`.
- Funkcja zadeklarowana w bloku `if` — zachowanie `var`-podobne/niespójne w starych silnikach; deklaruj funkcje na poziomie scope'u.

## Pytania rekrutacyjne

1. **Czym jest closure? Podaj praktyczny przykład.** — funkcja + środowisko leksykalne; licznik/memoize/debounce.
2. **Co wypisze kod?**
   ```js
   for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);
   ```
   — `3 3 3`; napraw przez `let`, IIFE albo argument funkcji. Bonus: powiąż z [event loopem](./09-event-loop.md).
3. **Czym różni się hoisting `var`, `let` i deklaracji funkcji?** — undefined vs TDZ vs całe ciało.
4. **Jak uzyskać prywatne zmienne bez klas?** — closure/module pattern; dziś też `#pola` ([obiekty i prototypy](./05-obiekty-i-prototypy.md)).
5. **Jak closures mogą powodować memory leaki?** — długowieczne referencje do domkniętych struktur/DOM.

## Dalsza lektura

- [MDN: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- [javascript.info: Variable scope, closure](https://javascript.info/closure)
- „You Don't Know JS Yet: Scope & Closures" (K. Simpson)
