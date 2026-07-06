# Funkcje i generyki

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Interfejsy i typy](./02-interfejsy-i-typy.md)

Generyki to zmienne dla typów: pozwalają pisać kod działający na wielu typach **bez utraty informacji o typie** (w przeciwieństwie do `any`). To granica, za którą „umiem TS" zaczyna znaczyć „umiem projektować API".

## Typowanie funkcji

```ts
function fn(a: number, b = 10, ...rest: string[]): number { … }
const cb: (id: number) => void = …;
type Handler = (e: Event) => void;

interface Search {
  (query: string): Result[];        // call signature
  cache: Map<string, Result[]>;     // + właściwości (funkcje to obiekty)
}
```

- Parametry opcjonalne `?` muszą być po wymaganych; domyślne dają inference.
- Zwrot `void` w typie callbacku ⇒ implementacja może zwracać cokolwiek (celowa elastyczność — `arr.forEach(x => arr2.push(x))` działa).
- **Overloads** — kilka sygnatur nad jedną implementacją; używaj rzadko, często unia/generyk wystarcza:

```ts
function parse(x: string): object;
function parse(x: Buffer): object;
function parse(x: string | Buffer): object { … }   // sygnatura implementacji niewidoczna
```

## Generyki — podstawy

```ts
function first<T>(arr: T[]): T | undefined { return arr[0]; }
first([1, 2, 3]);        // T = number — WYWNIOSKOWANE z argumentu
first<string>([]);        // jawnie, gdy nie ma z czego wnioskować

interface ApiResponse<T> { data: T; status: number; }
type Nullable<T> = T | null;
class Store<S> { constructor(private state: S) {} }
```

**Sens generyka = powiązanie typów** (wejścia z wyjściem, dwóch parametrów). Generyk użyty **raz** w sygnaturze to zwykle zbędny generyk (mogłby być konkretnym typem/unknown).

## Constraints — `extends`

```ts
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}
longest('abc', 'de');    // ✔ string ma length
longest(10, 20);          // ✘ number nie ma

function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];        // typ zwrotu zależny od klucza!
}
getProp(user, 'name');    // string
getProp(user, 'nope');    // ✘ błąd kompilacji
```

`K extends keyof T` + indexed access `T[K]` to najczęstszy „zaawansowany" idiom w praktyce (typowane gettery, formularze, tabele, i18n).

**Wartości domyślne:** `interface Props<T = unknown> { … }`.

## Generyki w praktyce frontendowej

```ts
// Typowana warstwa API (fasada z pliku o fetch)
async function api<T>(path: string): Promise<T> { … }
const users = await api<User[]>('/users');

// Typowany event emitter (wzorce projektowe — observer)
interface Events { 'cart:add': { id: number }; 'auth:logout': void; }
function emit<K extends keyof Events>(event: K, payload: Events[K]) { … }
emit('cart:add', { id: 1 });      // payload sprawdzony per event!

// Kolekcje/utilsy
function groupBy<T, K extends PropertyKey>(items: T[], key: (item: T) => K): Record<K, T[]> { … }
```

Uwaga do `api<T>`: `T` to **deklaracja zaufania**, nie walidacja — odpowiedź może łamać typ w runtime ([walidacja: TS w praktyce](./05-typescript-w-praktyce.md)).

## Inference w głąb — co warto wiedzieć

- TS wnioskuje z **miejsca użycia** (contextual typing): callback `arr.map(x => …)` zna typ `x` z tablicy.
- `const` zawęża do literałów; `as const` zamraża strukturę do literałów i readonly ([podstawy](./01-podstawy-typowania.md)).
- Generyki „przepływają": `Promise.all([fetchA(), fetchB()])` → tuple typów wyników ([asynchroniczność](../04-javascript/08-asynchronicznosc.md)).
- Zmienność (variance): funkcje są kontrawariantne w parametrach — stąd zakazy, które czasem zaskakują przy przypisywaniu callbacków; wystarczy kojarzyć zjawisko.

## Pułapki i częste błędy

- `<T>` dla ozdoby (użyty raz) albo `T extends any`.
- Zwracanie `any` z funkcji generycznych — cały zysk przepada; sprawdzaj, co naprawdę wnioskuje się na końcach.
- Overloads tam, gdzie wystarczy unia parametru lub generyk.
- Zbyt sprytne typy w kodzie aplikacyjnym — jeśli sygnatury wymagają komentarza-eseju, uprość; skomplikowane typy należą do bibliotek/utils.
- Mylenie generyka funkcji (per wywołanie) z generykiem typu/klasy (per instancja).

## Pytania rekrutacyjne

1. **Po co generyki, skoro jest `any`?** — zachowanie informacji o typie i powiązań między parametrami.
2. **Wyjaśnij `K extends keyof T` i `T[K]`.** — bezpieczny dostęp po kluczu; typ zwrotu zależny od argumentu.
3. **Kiedy generyk jest zbędny?** — pojedyncze użycie; heurystyka projektowania API.
4. **Napisz typ funkcji `pick(obj, keys)` zachowującej typy.** — `<T, K extends keyof T>(obj: T, keys: K[]) => Pick<T, K>` ([utility types](./04-typy-zaawansowane.md)).
5. **Czym są overloads i jakie mają ograniczenia?** — wiele sygnatur, implementacja niesprawdzana per overload; alternatywy.

## Dalsza lektura

- [TS Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [TS Handbook: More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)
- [Total TypeScript: Generics — kurs/artykuły](https://www.totaltypescript.com/)
