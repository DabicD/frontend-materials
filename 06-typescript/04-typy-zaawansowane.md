# Typy zaawansowane: mapped, conditional, utility types

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Funkcje i generyki](./03-funkcje-i-generyki.md)

Poziom „type-level programming": typy liczone z innych typów. W kodzie aplikacyjnym używasz gotowych utility types; umiejętność ich **zbudowania samemu** to sprawdzian rozumienia systemu typów (i ulubione zadanie rekrutacyjne: „zaimplementuj Pick").

## Mapped types — pętla po kluczach

```ts
type Readonly<T>  = { readonly [K in keyof T]: T[K] };
type Partial<T>   = { [K in keyof T]?: T[K] };
type Flags<T>     = { [K in keyof T]: boolean };

// modyfikatory +/-
type Mutable<T>   = { -readonly [K in keyof T]: T[K] };
type Required<T>  = { [K in keyof T]-?: T[K] };

// remapowanie kluczy przez `as` (+ template literal types)
type Getters<T> = { [K in keyof T as `get${Capitalize<K & string>}`]: () => T[K] };
// Getters<{name: string}> → { getName: () => string }
```

## Conditional types i `infer`

Warunek na poziomie typów: `T extends U ? X : Y`.

```ts
type IsArray<T> = T extends unknown[] ? true : false;

// infer — "wyłuskaj" typ z pozycji wzorca
type ElementOf<T>  = T extends (infer E)[] ? E : never;
type ReturnType<T> = T extends (...a: never[]) => infer R ? R : never;
type Awaited<T>    = T extends Promise<infer U> ? Awaited<U> : T;   // rekursja!
```

**Dystrybutywność:** warunek na „gołym" parametrze typu rozdziela się po unii — `Exclude<'a'|'b', 'a'>` liczy się per wariant i daje `'b'`. Wyłączenie: opakuj w tuple `[T] extends [U]`.

## Template literal types

```ts
type EventName = `on${Capitalize<'click' | 'focus'>}`;   // 'onClick' | 'onFocus'
type Route = `/users/${number}`;                          // pasuje '/users/42'
type PropPath<T> = …                                       // typowane ścieżki 'a.b.c' (biblioteki formularzy)
```

Silnik typowanych route'ów, tłumaczeń i CSS-in-TS w popularnych bibliotekach.

## Utility types — ściąga (i co mają w środku)

| Typ | Działanie | Implementacja (esencja) |
|---|---|---|
| `Partial<T>` / `Required<T>` | wszystko opcjonalne / wymagane | mapped + `?`/`-?` |
| `Readonly<T>` | pola readonly | mapped + `readonly` |
| `Pick<T, K>` | podzbiór pól | `{ [P in K]: T[P] }`, `K extends keyof T` |
| `Omit<T, K>` | bez pól K | `Pick<T, Exclude<keyof T, K>>` |
| `Record<K, V>` | słownik | `{ [P in K]: V }` |
| `Exclude<T, U>` / `Extract<T, U>` | filtr unii | dystrybutywny conditional |
| `NonNullable<T>` | bez null/undefined | `T & {}` |
| `ReturnType<F>` / `Parameters<F>` | z sygnatury funkcji | `infer` |
| `Awaited<T>` | rozpakowany Promise | rekurencyjny conditional |

Praktyka aplikacyjna — typy pochodne zamiast dublowania:

```ts
type UserUpdate = Partial<Omit<User, 'id'>>;          // payload PATCH
type UserFormErrors = Partial<Record<keyof User, string>>;
type Handlers = { [K in Status]: () => void };         // wymusza obsługę każdego statusu
```

## Trzy idiomy senior-level

**1. `as const` + `typeof` — typ ze stałej (jedno źródło prawdy):**

```ts
const CONFIG = { locales: ['pl', 'en'], defaultLocale: 'pl' } as const;
type Locale = typeof CONFIG.locales[number];   // 'pl' | 'en'
```

**2. Branded types — nominalność w strukturalnym świecie:**

```ts
type UserId = number & { readonly __brand: 'UserId' };
type OrderId = number & { readonly __brand: 'OrderId' };
// funkcja przyjmująca UserId nie przyjmie OrderId, mimo że oba to number
```

Chroni przed pomieszaniem identyfikatorów/jednostek — zero kosztu runtime.

**3. Typowanie granic zamiast wnętrza:** wyprowadzaj typy z walidatorów (`z.infer` — [TS w praktyce](./05-typescript-w-praktyce.md)), ze schematu API (codegen OpenAPI/GraphQL — [GraphQL](../12-komunikacja-z-api/02-graphql.md)) — ręcznie pisane typy odpowiedzi rozjeżdżają się z rzeczywistością.

## Pułapki i częste błędy

- Type-level golf w kodzie produktowym — typ, którego nikt nie rozumie, jest gorszy niż prostszy i trochę mniej precyzyjny.
- Zapominanie o dystrybutywności — `T extends U` na unii robi „mapę", nie pojedynczy test.
- `Omit` z literówką w kluczu **nie protestuje** (K extends PropertyKey, nie keyof T) — pisz własny `StrictOmit` lub uważaj.
- Głęboka rekursja typów — limity kompilatora i czas kompilacji (patrz „Type instantiation is excessively deep").
- Utrzymywanie ręcznych typów duplikujących schematy — generuj/wyprowadzaj.

## Pytania rekrutacyjne

1. **Zaimplementuj `Pick`/`Partial`/`ReturnType` od zera.** — mapped types / infer; klasyk tablicowy.
2. **Co robi `infer`?** — wiąże typ z pozycji wzorca w conditional type.
3. **Czym jest dystrybutywność conditional types?** — rozdzielanie po unii; jak wyłączyć.
4. **Jak uzyskać typ z tablicy `as const`?** — `typeof arr[number]`.
5. **Do czego służą branded types?** — rozróżnienie nominalne identycznych prymitywów (id, jednostki).

## Dalsza lektura

- [TS Handbook: Mapped / Conditional / Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
- [type-challenges](https://github.com/type-challenges/type-challenges) — ćwiczenia (zrób poziom easy+medium przed rozmowami)
