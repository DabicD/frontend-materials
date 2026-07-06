# JavaScript zaawansowany: generatory, Symbol, Proxy, pamięć

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Kolekcje i iteracja](./06-kolekcje-i-iteracja.md), [Obiekty i prototypy](./05-obiekty-i-prototypy.md), [Closures](./03-scope-hoisting-closures.md)

Tematy, które rzadko piszesz ręcznie, ale które napędzają biblioteki, których używasz codziennie (reaktywność Vue/MobX = Proxy, Redux-Saga = generatory) — i które odróżniają seniora przy debugowaniu problemów z pamięcią.

## Generatory

Funkcja, którą można **wstrzymywać i wznawiać**; każde `yield` oddaje wartość i pauzuje:

```js
function* idGenerator() {
  let id = 1;
  while (true) yield id++;
}
const gen = idGenerator();
gen.next();   // { value: 1, done: false } — leniwe: liczy dopiero na żądanie

function* take(iterable, n) {
  let i = 0;
  for (const item of iterable) {
    if (i++ >= n) return;
    yield item;
  }
}
[...take(idGenerator(), 3)];   // [1, 2, 3] — kompozycja leniwych sekwencji
```

- Generator **jest iteratorem i iterable** — najprostszy sposób implementacji `[Symbol.iterator]`.
- Komunikacja dwustronna: `gen.next(value)` wstrzykuje wartość jako wynik `yield` (na tym stały biblioteki efektów typu Redux-Saga).
- `yield*` — delegacja do innego generatora.
- **Async generatory** (`async function*` + `for await...of`) — strumienie: paginacja API, czytanie response body kawałkami.
- Nowe Iterator helpers (`.map`, `.filter`, `.take` na iteratorach) robią leniwe przetwarzanie natywnie.

## Symbol

Unikalny, niekolidujący klucz właściwości:

```js
const META = Symbol('meta');
obj[META] = { … };            // niewidoczny dla Object.keys / JSON.stringify / for...in
```

**Well-known symbols** — „protokoły" języka, przez które obiekt integruje się z mechanizmami JS: `Symbol.iterator` ([iteracja](./06-kolekcje-i-iteracja.md)), `Symbol.asyncIterator`, `Symbol.toPrimitive` (własna konwersja), `Symbol.toStringTag`, `Symbol.hasInstance` (własne `instanceof`).

## Proxy i Reflect

**Proxy** przechwytuje operacje na obiekcie (trap-y: `get`, `set`, `has`, `deleteProperty`, `apply`…):

```js
const state = new Proxy({ count: 0 }, {
  get(target, prop, receiver) {
    track(prop);                                  // "ktoś czyta count" → zapisz zależność
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    const ok = Reflect.set(target, prop, value, receiver);
    trigger(prop);                                // "count się zmienił" → odpal efekty
    return ok;
  },
});
```

To dosłownie serce **reaktywności Vue 3 / MobX / Solid stores** — automatyczne śledzenie zależności bez jawnych subskrypcji ([modele reaktywności](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md)). Inne użycia: walidacja dostępu, API mock-ujące, negative array indices, deprecation warnings.

**Reflect** — metody odpowiadające 1:1 trap-om; używaj w trap-ach zamiast operacji bezpośrednich (poprawny `receiver`, spójne wartości zwrotne). Ograniczenia Proxy: identyczność (`proxy !== target`), problemy z private fields klas i niektórymi wbudowanymi (Date).

## Zarządzanie pamięcią i garbage collection

JS zwalnia pamięć automatycznie — **GC usuwa obiekty nieosiągalne** z korzeni (global, stack, aktywne closures). Algorytm: mark-and-sweep, generacyjny (młode obiekty sprzątane często i tanio — stąd „śmiecenie" krótkotrwałymi obiektami jest zwykle OK).

**Memory leak = obiekt osiągalny, choć niepotrzebny.** GC go NIE zabierze. Klasyczne źródła we frontendzie:

1. **Zapomniane subskrypcje/listenery:** `addEventListener` bez `removeEventListener` (gdy element żyje krócej niż target — np. listener na `window` z komponentu), `setInterval` bez `clearInterval`, niezamknięte WebSockety/observery. Wzorzec: każdy „mount" ma swój „cleanup" (`AbortController` dla listenerów — [zdarzenia](../05-dom-i-web-api/02-zdarzenia.md)).
2. **Oderwane węzły DOM (detached nodes)** trzymane w zmiennych/mapach po usunięciu z drzewa.
3. **Długowieczne closures** domykające wielkie struktury ([closures](./03-scope-hoisting-closures.md)).
4. **Rosnące cache bez limitu** — używaj `WeakMap`/`WeakRef` + LRU.
5. Globalne kolekcje „na chwilę", do których tylko się dopisuje.

`WeakRef` i `FinalizationRegistry` — słabe referencje na żądanie (zaawansowane cache); używaj rzadko i ostrożnie.

**Diagnostyka:** DevTools → Memory: porównanie heap snapshots (co przybyło), allocation timeline, szukanie „Detached" — warsztat: [DevTools](../14-narzedzia/05-devtools.md). Objaw leaka: pamięć karty rośnie z użyciem i nie spada po akcjach „sprzątających".

## Drobiazgi seniora

- **Strict mode** — moduły i klasy mają go zawsze; wiedz, co zmienia (this=undefined, błędy zamiast cichych ignorów).
- **Tagged unions / obiekty z polem `type`** — silnik optymalizuje obiekty o stałym kształcie (hidden classes); unikaj dopisywania pól „po czasie" w hot path.
- `structuredClone`, `Intl.*` (formatowanie dat/liczb/walut — nie pisz własnego!), `AbortController` jako uniwersalny wzorzec anulowania.

## Pułapki i częste błędy

- Traktowanie GC jak magii — „przecież JS sam zwalnia" ≠ brak leaków; leak to problem **osiągalności**, nie języka.
- Listener na `window`/`document` dodawany przy każdym renderze komponentu.
- Cache w zwykłej `Map` z kluczami-obiektami — trzyma klucze wiecznie (użyj `WeakMap`).
- Nadużywanie Proxy tam, gdzie wystarczą gettery — koszt wydajności i debugowalności.
- Generatory jako „fajny trick" w kodzie zespołowym bez potrzeby — czytelność.

## Pytania rekrutacyjne

1. **Jak działa garbage collection w JS?** — osiągalność od korzeni, mark-and-sweep, generacyjność.
2. **Podaj 3 typowe źródła memory leaków we frontendzie i jak je wykryć.** — listenery/intervale, detached DOM, cache/closures; heap snapshots.
3. **Do czego frameworki używają Proxy?** — śledzenie odczytów/zapisów → automatyczna reaktywność.
4. **Czym generator różni się od zwykłej funkcji?** — pauzowanie/wznawianie, leniwość, iterator+iterable, komunikacja przez next(v).
5. **`WeakMap` vs `Map` w kontekście pamięci?** — słabe klucze nie blokują GC; cache/metadane per obiekt.

## Dalsza lektura

- [javascript.info: Generators](https://javascript.info/generators), [Proxy and Reflect](https://javascript.info/proxy)
- [MDN: Memory management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management)
- [Chrome DevTools: Fix memory problems](https://developer.chrome.com/docs/devtools/memory-problems)
