# Wzorce projektowe w JavaScript

> **Poziom:** 🟡 średni → 🔴 zaawansowany
> **Wymagana wiedza:** [Funkcje](./02-funkcje.md), [Closures](./03-scope-hoisting-closures.md), [Moduły](./10-moduly.md)

Wzorce projektowe to nazwane, sprawdzone rozwiązania powtarzalnych problemów. W JS klasyczne wzorce GoF często upraszczają się do funkcji i modułów — wartością nie jest recytowanie katalogu, lecz **rozpoznawanie wzorca w API bibliotek** i dobieranie go do problemu. (Wzorce architektury całej aplikacji: [struktura projektu](../08-architektura-aplikacji/04-struktura-projektu.md); wzorce frameworkowe: [zarządzanie stanem](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)).

## Module i Revealing Module

Prywatny stan + publiczne API — dziś naturalnie przez [moduły ESM](./10-moduly.md) + [closures](./03-scope-hoisting-closures.md):

```js
// cart.js — wszystko poza exportami jest prywatne
let items = [];
export const cart = {
  add: item => { items = [...items, item]; },
  total: () => items.reduce((s, i) => s + i.price, 0),
};
```

## Singleton

Jedna instancja współdzielona globalnie. W JS **moduł jest singletonem z natury** (wykonuje się raz):

```js
export const apiClient = createClient(config);   // wszyscy importują tę samą instancję
```

Świadomie: singleton to globalny stan — utrudnia testy (potrzebne resetowanie/mocki) i wiąże moduły niejawnie. Alternatywa: **dependency injection** — przekazuj zależność jawnie (argumentem/parametrem), a singleton trzymaj na brzegu aplikacji (composition root).

## Observer / Pub-Sub

Obiekt powiadamia subskrybentów o zmianach — fundament eventów DOM, streamów, store'ów i [reaktywności](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md):

```js
function createEmitter() {
  const listeners = new Map();   // event → Set<fn>
  return {
    on(event, fn) {
      if (!listeners.has(event)) listeners.set(event, new Set());
      listeners.get(event).add(fn);
      return () => listeners.get(event).delete(fn);   // unsubscribe! (leaki)
    },
    emit(event, data) { listeners.get(event)?.forEach(fn => fn(data)); },
  };
}
```

**Observer** — subskrybent zna źródło (`store.subscribe(fn)`); **Pub-Sub** — pełne odsprzęgnięcie przez pośrednika (event bus). Event bus w dużej aplikacji = przepływ trudny do śledzenia; preferuj jawny stan.

## Factory i Builder

**Factory** — funkcja tworząca obiekty, ukrywająca szczegóły konstrukcji (w JS zwykle zamiast `new` i hierarchii klas):

```js
const createUser = (data) => ({ ...defaults, ...data, id: crypto.randomUUID() });
const createLogger = (env) => env === 'prod' ? sentryLogger : consoleLogger;  // wybór wariantu
```

**Builder** — konstrukcja krokowa przez fluent API (`query.where(…).orderBy(…).limit(5)`); spotykany w bibliotekach (ORM-y, konfiguracje), rzadko pisany ręcznie.

## Strategy i State

**Strategy** — wymienne algorytmy pod wspólnym interfejsem. W JS: obiekt-mapa funkcji zamiast łańcucha if/else:

```js
const shippingStrategies = {
  courier: order => 15,
  pickup: () => 0,
  express: order => order.total > 200 ? 20 : 35,
};
const cost = shippingStrategies[order.shipping](order);
```

**State / maszyny stanów** — zachowanie zależne od jawnego stanu z zdefiniowanymi przejściami. Antidotum na „boolean soup" (`isLoading && !isError && …`): stany `idle | loading | success | error` jako unia — [wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md), formalnie: XState.

## Decorator / wrapper i middleware

Rozszerzanie zachowania bez zmiany oryginału — funkcje wyższego rzędu:

```js
const withRetry = (fn, times = 3) => async (...args) => {
  for (let i = 0; ; i++) {
    try { return await fn(...args); }
    catch (e) { if (i >= times - 1) throw e; }
  }
};
const fetchUser = withRetry(withLogging(rawFetchUser));
```

**Middleware/pipeline** (Express, Redux, fetch interceptory) to złożenie takich wrapperów w łańcuch.

## Adapter i Facade

- **Adapter** — tłumaczy obcy interfejs na Twój (np. warstwa nad API płatności: `stripeAdapter`, `payuAdapter` z jednym kontraktem).
- **Facade** — proste API nad złożonym podsystemem (Twój moduł `api.js` nad fetch+auth+retry to fasada).

Oba wzorce to praktyczna **warstwa antykorupcyjna**: izolują aplikację od zewnętrznych bibliotek — wymiana biblioteki dotyka jednego pliku ([czysty kod](../15-inzynieria-oprogramowania/01-czysty-kod.md)).

## Kompozycja funkcji

Deklaratywne budowanie potoków z małych czystych funkcji:

```js
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);
const slugify = pipe(trim, toLowerCase, replaceSpaces);
```

## Pułapki i częste błędy

- Wzorzec dla wzorca (patternitis) — fabryka fabryk tam, gdzie wystarczy funkcja; wzorzec ma **usuwać** złożoność.
- Observer bez mechanizmu odsubskrybowania — memory leak ([pamięć](./12-zaawansowane.md)).
- Ukryte singletony (moduł z mutowalnym stanem) — testy zależne od kolejności; jawne DI na brzegach.
- Event bus jako główna architektura komunikacji — spaghetti zdarzeń.
- Kopiowanie wzorców klasowych z Javy 1:1 — w JS funkcje + closures załatwiają większość prościej.

## Pytania rekrutacyjne

1. **Jakie wzorce stosujesz w praktyce frontendowej?** — module, observer (store), strategy (mapy funkcji), adapter/fasada na API, middleware; przykłady z bibliotek.
2. **Zaimplementuj prosty event emitter.** — on/emit/unsubscribe (patrz wyżej) — częste zadanie live coding.
3. **Singleton — plusy, minusy, jak testować kod z singletonem?** — wygoda vs globalny stan; DI, reset w testach.
4. **Czym różni się Observer od Pub-Sub?** — bezpośrednia subskrypcja vs pośrednik; kompromisy śledzenia przepływu.
5. **Jak Strategy eliminuje if/else?** — mapa wariantów + wspólny interfejs; otwarte na rozszerzenie.

## Dalsza lektura

- [Patterns.dev](https://www.patterns.dev/) — wzorce w kontekście nowoczesnego frontendu (za darmo)
- [Refactoring.guru: Design Patterns](https://refactoring.guru/design-patterns) — katalog z diagramami
- „Learning JavaScript Design Patterns" — A. Osmani (online, aktualizowana)
