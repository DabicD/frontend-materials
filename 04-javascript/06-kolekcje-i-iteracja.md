# Kolekcje i iteracja

> **Poziom:** 🟢 podstawowy → 🟡 średni (iterators, Weak*)
> **Wymagana wiedza:** [Funkcje](./02-funkcje.md), [Obiekty](./05-obiekty-i-prototypy.md)

Tablice i ich metody to codzienny warsztat — transformacje danych z API do widoku. Do tego protokół iteracji (wspólny język pętli, spreadu i destrukturyzacji) oraz `Map`/`Set`, które zbyt często zastępowane są „obiektem na wszystko".

## Metody tablic — podział, który porządkuje

**Transformujące (zwracają NOWĄ tablicę / wartość — nie mutują):**

```js
arr.map(fn)                    // 1:1 transformacja
arr.filter(fn)                 // podzbiór
arr.reduce((acc, x) => …, init)   // dowolna agregacja (suma, grupowanie, obiekt)
arr.flat(depth), arr.flatMap(fn)
arr.slice(od, do)              // wycinek (do wyłącznie)
arr.concat(b), [...a, ...b]
// nowoczesne niemutujące odpowiedniki starych mutatorów:
arr.toSorted(cmp), arr.toReversed(), arr.toSpliced(...), arr.with(i, val)
```

**Wyszukujące:**

```js
arr.find(fn), arr.findIndex(fn), arr.findLast(fn)
arr.includes(x)                // uwaga: === (obiektu nie znajdzie po zawartości)
arr.indexOf(x)
arr.some(fn), arr.every(fn)    // czy istnieje / czy wszystkie
arr.at(-1)                     // ostatni element
```

**Mutujące (zmieniają oryginał!):** `push/pop`, `shift/unshift`, `splice`, `sort`, `reverse`, `fill`, `copyWithin`.

**Dwie sławne pułapki:**
- `sort()` bez komparatora sortuje **jako stringi**: `[10, 9, 1].sort()` → `[1, 10, 9]`. Liczby: `sort((a, b) => a - b)`.
- `map/filter` nie mutują, ale **obiekty w środku to wciąż te same referencje** — „głęboka" zmiana wymaga kopiowania elementów.

**`reduce` — umiej płynnie:**

```js
const total = items.reduce((sum, i) => sum + i.price, 0);
const byId = users.reduce((acc, u) => ({ ...acc, [u.id]: u }), {});
// (grupowanie ma już dedykowane API: Object.groupBy(users, u => u.role))
```

## Protokół iteracji

**Iterable** = obiekt z metodą `[Symbol.iterator]()` zwracającą **iterator** (`{ next(): { value, done } }`). Iterable są: tablice, stringi, `Map`, `Set`, `NodeList`, `arguments`… **Obiekt `{}` NIE jest iterable.**

Z protokołu korzystają: `for...of`, spread `[...x]`, destrukturyzacja tablicowa, `Array.from`, `Promise.all`, `new Map(iterable)`.

```js
const range = {
  from: 1, to: 3,
  [Symbol.iterator]() {
    let cur = this.from, last = this.to;
    return { next: () => cur <= last ? { value: cur++, done: false } : { value: undefined, done: true } };
  },
};
[...range];   // [1, 2, 3]
```

(Wygodniejsza implementacja iteratorów — generatory: [zaawansowane](./12-zaawansowane.md).)

**Pętle — którą wybrać:**
- `for...of` — wartości iterable (+ `break`/`continue`; async wariant `for await...of`).
- `for...in` — **klucze obiektu** (w tym odziedziczone enumerowalne!). Nie dla tablic.
- `forEach` — wygodny, ale bez `break` i **nie czeka na `await`** ([asynchroniczność](./08-asynchronicznosc.md)).
- Po obiektach: `Object.keys/values/entries(obj)` + `for...of`.

## Map i Set

**`Map`** — słownik z kluczami **dowolnego typu** (obiekty, funkcje!), z gwarantowaną kolejnością wstawiania i `size`:

```js
const m = new Map([[userObj, 'meta']]);
m.get(userObj); m.set(k, v); m.has(k); m.delete(k); m.size;
```

Kiedy `Map` zamiast obiektu: klucze nieznane z góry/dynamiczne, klucze nie-stringowe, częste dodawanie/usuwanie, potrzebny `size`/iteracja. Obiekt: stała, znana struktura („rekord").

**`Set`** — kolekcja unikalnych wartości (porównanie jak `===`):

```js
const unique = [...new Set(arr)];         // deduplikacja — idiom
set.has(x);                                // O(1) — szybkie "czy zawiera"
// operacje mnogościowe: a.intersection(b), a.union(b), a.difference(b)
```

**`WeakMap` / `WeakSet`** — klucze/elementy tylko obiektowe i trzymane **słabo**: nie blokują [garbage collectora](./12-zaawansowane.md). Nieiterowalne, bez `size`. Zastosowania: metadane przypięte do cudzych obiektów (np. DOM node → dane), cache per obiekt, prywatne dane — wszystko bez ryzyka memory leaka.

## Array-like i konwersje

`NodeList` z `querySelectorAll` czy `arguments` bywają „array-like" (mają `length`, nie mają metod tablicy — [DOM](../05-dom-i-web-api/01-dom-manipulacja.md)):

```js
Array.from(nodeList)            // konwersja (+ opcjonalny map: Array.from(x, fn))
Array.from({ length: 5 }, (_, i) => i)   // [0..4]
[...iterable]                   // tylko dla iterable
```

## Pułapki i częste błędy

- `sort` liczb bez komparatora; `sort`/`reverse` mutują oryginał (użyj `toSorted`/`toReversed`).
- `includes/indexOf` dla obiektów — porównanie referencji; szukaj przez `find`/`some`.
- `for...in` po tablicy — klucze jako stringi + odziedziczone; używaj `for...of`.
- Mutowanie stanu przez `push`/`splice` tam, gdzie framework oczekuje nowej referencji ([reaktywność](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md)).
- `new Array(3)` — tablica z dziurami (empty slots), po której `map` nie iteruje; `Array.from({length: 3})`.
- Budowanie obiektu w `reduce` przez spread akumulatora w każdej iteracji — O(n²) na dużych danych; mutuj akumulator lokalnie albo użyj `Object.fromEntries`/`Object.groupBy`.

## Pytania rekrutacyjne

1. **`map` vs `forEach`?** — nowa tablica vs efekt uboczny; forEach nie łamie się breakiem i ignoruje await.
2. **Kiedy `Map` zamiast zwykłego obiektu?** — dowolne typy kluczy, size, kolejność, częste mutacje.
3. **Do czego służy `WeakMap`?** — słabe referencje: metadane/cache bez blokowania GC.
4. **Co musi spełniać obiekt, żeby działał z `for...of` i spreadem?** — protokół iteracji: Symbol.iterator → iterator z next().
5. **Dlaczego `[10, 9, 1].sort()` daje `[1, 10, 9]`?** — domyślne porównanie leksykograficzne stringów.
6. **Zaimplementuj deduplikację tablicy obiektów po `id`.** — Map po id / Set widzianych id + filter.

## Dalsza lektura

- [MDN: Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) — przejrzyj WSZYSTKIE metody raz
- [javascript.info: Iterables](https://javascript.info/iterable), [Map and Set](https://javascript.info/map-set)
