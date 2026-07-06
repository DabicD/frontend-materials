# TypeScript — podstawy typowania

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** [Typy w JS](../04-javascript/01-typy-i-zmienne.md)

TypeScript = JavaScript + statyczny system typów, **wymazywany przy kompilacji** (type erasure): typy nie istnieją w runtime — to narzędzie analizy i dokumentacji, nie walidacji danych. TS jest dziś standardem zawodowym; „znajomość TS" na poziomie senior oznacza swobodę czytania i projektowania typów, nie tylko dopisywania `: string`.

## Anotacje i inference

```ts
let count = 0;                    // inference: number — NIE pisz ": number" tu
const name = 'Ala';               // inference: literal 'Ala' (const → zawężenie)
let user: User;                   // anotacja, gdy brak wartości początkowej

function total(items: Item[], tax: number): number { … }
//            ^ parametry ZAWSZE anotuj    ^ return zwykle wyinferowany
```

**Zasada:** pozwól TS wnioskować, anotuj **granice**: parametry funkcji, publiczne API modułów, miejsca, gdzie inference daje za szeroki typ. Nadmiarowe anotacje to szum i okazja do rozjazdu.

## Typy podstawowe

```ts
string, number, boolean, bigint, symbol, null, undefined
number[]  /  Array<number>              // tablice
[number, string]                         // tuple — stała długość i typy pozycji
[lat: number, lng: number]               // nazwane elementy tuple (czytelność)
{ id: number; name?: string }            // obiekt; ? = opcjonalne (T | undefined)
readonly id: number / readonly number[]  // tylko-do-odczytu (compile time!)
(a: number, b: number) => number         // typ funkcji
```

## `any`, `unknown`, `never`, `void`

- **`any`** — wyłącza sprawdzanie; **zaraża** (operacje na any dają any). Każdy `any` to dziura w systemie typów. Lint: `no-explicit-any`.
- **`unknown`** — bezpieczny odpowiednik: „nie wiem, co to jest" — **nie można nic z tym zrobić bez zawężenia**. Właściwy typ dla danych z zewnątrz (API, JSON.parse, catch):

```ts
function handle(err: unknown) {
  if (err instanceof Error) console.error(err.message);   // najpierw sprawdź
}
```

- **`never`** — typ „to się nie zdarza": funkcje rzucające/nieskończone, wyczerpane unie (exhaustiveness — [interfejsy i typy](./02-interfejsy-i-typy.md)).
- **`void`** — funkcja nic nie zwraca (callbacki `() => void` akceptują dowolny zwrot — celowo).

## Strict mode — nienegocjowalny

`"strict": true` w tsconfig włącza m.in.:

- **`strictNullChecks`** — `null`/`undefined` nie pasują do `string` po cichu; TS zmusza do obsługi braków. To największa wartość TS: klasa błędów „cannot read property of undefined" wykrywana w edytorze.
- `noImplicitAny` — parametry bez typu nie stają się cicho `any`.

```ts
function len(s?: string) {
  s.length;      // ✘ błąd: s może być undefined
  return s?.length ?? 0;   // ✔ obsłużone (operatorami z ES6+)
}
```

TS bez strict to złudzenie bezpieczeństwa. Nowy projekt: strict zawsze; legacy: włączaj flagami stopniowo ([TS w praktyce](./05-typescript-w-praktyce.md)).

## Enum vs unie literałów

```ts
enum Status { Active, Inactive }          // enum: generuje KOD w runtime
type Status = 'active' | 'inactive';      // unia literałów: zero runtime, prostsza
```

Współczesna praktyka: **preferuj unie literałów** (lub `as const` obiekty) — bez kodu w runtime, lepsze inference, bez dziwactw enum. Enum spotkasz w istniejących codebase'ach.

```ts
const ROLES = ['admin', 'editor', 'viewer'] as const;
type Role = typeof ROLES[number];         // 'admin' | 'editor' | 'viewer' — z tablicy!
```

## Asercje typów — używaj świadomie

```ts
const input = document.querySelector('#email') as HTMLInputElement;  // "wiem lepiej"
const cfg = {} as Config;        // ⚠ kłamstwo — pusty obiekt "udaje" Config
x!                                // non-null assertion: "na pewno nie null"
satisfies Config                  // sprawdź zgodność BEZ zmiany wyinferowanego typu
```

`as` **nie konwertuje** — tylko ucisza kompilator; błędna asercja = runtime crash z czystym sumieniem TS. `satisfies` to bezpieczniejszy krewny: waliduje, zachowując precyzyjny typ. Dane z zewnątrz waliduj w runtime (Zod itd. — [TS w praktyce](./05-typescript-w-praktyce.md)), nie asercją.

## Pułapki i częste błędy

- `any` jako szybka naprawa błędu kompilacji — dziura rozlewa się po projekcie; użyj `unknown` + zawężenie.
- Anotowanie wszystkiego (walka z inference) albo niczego (API modułów bez kontraktów).
- Wiara, że typy chronią w runtime — odpowiedź z API może łamać typ; granice systemu wymagają walidacji.
- `!` (non-null) rozsiane po kodzie — każde to potencjalny crash; zawężaj zamiast zapewniać.
- `readonly`/`private` traktowane jak zabezpieczenie runtime — to tylko compile time (wyjątek: `#pola` z [klas JS](../04-javascript/05-obiekty-i-prototypy.md)).

## Pytania rekrutacyjne

1. **`any` vs `unknown`?** — wyłączenie sprawdzania vs typ wymagający zawężenia; kiedy unknown.
2. **Co robi `strictNullChecks` i czemu jest ważny?** — null/undefined jawnie w typach; eliminuje klasę błędów runtime.
3. **Czy typy TS istnieją w runtime? Konsekwencje?** — erasure; walidacja danych zewnętrznych osobno.
4. **Enum czy unia literałów?** — kompromisy; skąd preferencja dla unii.
5. **Czym różni się `as` od `satisfies`?** — nadpisanie typu vs walidacja z zachowaniem inference.

## Dalsza lektura

- [TypeScript Handbook: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [Total TypeScript (M. Pocock) — darmowe tutoriale](https://www.totaltypescript.com/tutorials)
