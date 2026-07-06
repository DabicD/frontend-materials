# Interfejsy, unie i zawężanie typów

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Podstawy typowania](./01-podstawy-typowania.md)

Serce codziennego TS: modelowanie danych (`interface`/`type`), składanie typów (unie, przecięcia) i **narrowing** — sztuka prowadzenia kompilatora przez logikę tak, by typy zawężały się same. Dobrze zamodelowany typ czyni stany niemożliwe niereprezentowalnymi.

## `interface` vs `type`

```ts
interface User { id: number; name: string; }
interface Admin extends User { permissions: string[]; }   // rozszerzanie

type UserId = number | string;                 // alias dowolnego typu
type Admin2 = User & { permissions: string[] };  // intersection
```

Różnice praktyczne:
- `type` — może wszystko: unie, tuple, funkcje, mapped types. `interface` — tylko kształty obiektów.
- `interface` ma **declaration merging** (dwie deklaracje o tej samej nazwie się sklejają) — kluczowe do rozszerzania cudzych typów (np. globalnych), przypadkowe w kodzie własnym.
- Wydajnościowo (dla kompilatora) interface bywa lżejszy przy rozbudowanych hierarchiach.

**Praktyka:** wybierz konwencję zespołu i bądź spójny; częsta reguła: „interface dla publicznych kształtów obiektów, type do wszystkiego pozostałego". Różnica jest mniejsza niż waga spójności.

## Unie i przecięcia

```ts
type Status = 'idle' | 'loading' | 'success' | 'error';   // unia literałów
type Result = Data | null;
type Draggable = Positioned & Sized;                        // przecięcie: wszystkie pola
```

**Discriminated union — najważniejszy wzorzec modelowania w TS:**

```ts
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }        // data ISTNIEJE tylko w success
  | { status: 'error'; error: Error };    // error tylko w error

function render(state: FetchState<User[]>) {
  switch (state.status) {                  // pole-dyskryminator
    case 'success': return list(state.data);   // TS WIE, że data jest
    case 'error':   return oops(state.error);
    …
  }
}
```

Porównaj z „boolean soup" (`{ isLoading, isError, data?, error? }`) — tam nic nie broni przed `isLoading && data`. Ten wzorzec organizuje stany UI ([wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)) i redukuje testy „niemożliwych" kombinacji.

**Exhaustiveness check** — kompilator pilnuje obsługi wszystkich wariantów:

```ts
default: {
  const _exhaustive: never = state;   // błąd kompilacji, gdy dojdzie nowy wariant
  throw new Error('Unhandled');
}
```

## Narrowing — zawężanie typów

TS śledzi przepływ sterowania (control flow analysis) i zawęża typy po sprawdzeniach:

```ts
function fmt(x: string | number | null) {
  if (x === null) return '—';          // dalej: string | number
  if (typeof x === 'number') return x.toFixed(2);   // number
  return x.trim();                      // string — sam z eliminacji
}
```

Narzędzia zawężania: `typeof`, `instanceof`, `in` (`'error' in state`), porównania z literałami/null, `Array.isArray`, discriminator, przypisania.

**Type predicates (`is`)** — własne strażniki:

```ts
function isUser(x: unknown): x is User {
  return typeof x === 'object' && x !== null && 'id' in x;
}
if (isUser(data)) data.name;            // zawężone
```

- Uwaga: predicate to **obietnica bez dowodu** — błędna implementacja kłamie kompilatorowi (jak `as`).
- `asserts x is T` — wariant rzucający (assertion functions).
- Zawężanie nie przechodzi przez granice funkcji/callbacków (TS nie wie, kiedy callback się wykona) — stąd czasem lokalna stała: `const u = maybeUser; if (u) { arr.map(() => u.name) }`.

## Typy obiektowe — detale, które bolą

- **Structural typing:** zgodność po kształcie, nie nazwie — `{ id: 1, extra: true }` pasuje do `{ id: number }`. Wyjątek: **excess property check** dla świeżych literałów obiektowych (literał wprost w miejscu przypisania jest sprawdzany surowiej).
- **Opcjonalne `?` vs `| undefined`:** `a?: string` = można pominąć klucz; `a: string | undefined` = klucz musi być, wartość może być undefined (z `exactOptionalPropertyTypes` różnica jest realna).
- **Index signatures:** `Record<string, number>` / `{ [key: string]: number }` — słowniki; dostęp zwraca potencjalnie undefined z flagą `noUncheckedIndexedAccess` (włącz ją!).
- `keyof User` → unia nazw kluczy; `User['name']` → typ pola (indexed access). Fundament [typów zaawansowanych](./04-typy-zaawansowane.md).

## Pułapki i częste błędy

- Modelowanie stanów flagami boolean zamiast discriminated union.
- `interface` z polami opcjonalnymi „bo czasem nie ma" — często to sygnał, że są dwa różne typy (unia!).
- Structural typing zaskakuje: funkcja przyjmie „za szeroki" obiekt — to feature, nie bug (ale excess check literałów myli, bo działa inaczej).
- Type predicate zwracający `true` bez faktycznej walidacji.
- Zawężenie „gubione" w callbackach/async — snapshot do stałej.

## Pytania rekrutacyjne

1. **`interface` vs `type` — różnice i Twoja konwencja?** — możliwości, merging, spójność.
2. **Czym jest discriminated union i jaki problem rozwiązuje?** — stany wykluczające się; niemożliwe stany niereprezentowalne.
3. **Jak działa narrowing? Wymień sposoby.** — control flow + typeof/instanceof/in/dyskryminator/predicates.
4. **Co to structural typing?** — zgodność po kształcie; konsekwencje + excess property check.
5. **Jak wymusić obsługę wszystkich wariantów unii?** — never w default (exhaustiveness).

## Dalsza lektura

- [TS Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [TS Handbook: Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html)
- [Making impossible states impossible (koncept)](https://kentcdodds.com/blog/make-impossible-states-impossible)
