# Event loop

> **Poziom:** 🔴 zaawansowany (dom tematu — inne pliki tu linkują)
> **Wymagana wiedza:** [Asynchroniczność](./08-asynchronicznosc.md), [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)

Event loop to mechanizm, dzięki któremu jednowątkowy JavaScript obsługuje tysiące zdarzeń bez blokowania. To **pytanie-filtr na rozmowach senior**: zadania „co wypisze ten kod" testują dokładnie ten model. Opanuj go raz — poprawnie, z mikrotaskami.

## Elementy układanki

- **Call stack** — stos wywołań; JS wykonuje zawsze to, co na szczycie. Jedna rzecz naraz.
- **Web APIs** — timery, sieć, DOM eventy żyją **poza silnikiem JS** (w przeglądarce); tam „czekają".
- **Task queue (macrotask queue)** — kolejka gotowych callbacków: `setTimeout`/`setInterval`, eventy (klik), I/O, `postMessage`.
- **Microtask queue** — kolejka o wyższym priorytecie: callbacki `.then/.catch/.finally`, `queueMicrotask`, `MutationObserver`, kontynuacje po `await`.

**Algorytm pętli:**

```
weź 1 (mac­ro)task → wykonaj do końca
→ OPRÓŻNIJ CAŁĄ kolejkę mikrotasków (wraz z dokładanymi w trakcie!)
→ (może) rendering: rAF → style → layout → paint
→ wróć na początek
```

Dwie konsekwencje-klucze:

1. **Mikrotaski zawsze przed następnym macrotaskiem** — Promise "przeskakuje" setTimeout.
2. **Rendering dopiero między taskami** — długi task = zamrożony interfejs; pętla dokładająca mikrotaski w nieskończoność **zagłodzi** rendering totalnie.

## Kanoniczny przykład (naucz się rozpisywać)

```js
console.log('1');                                    // sync
setTimeout(() => console.log('2'), 0);               // macrotask
Promise.resolve().then(() => console.log('3'));      // microtask
queueMicrotask(() => console.log('4'));              // microtask
console.log('5');                                    // sync

// 1, 5, 3, 4, 2
```

Rozpiska: kod synchroniczny do końca (1, 5) → wszystkie mikrotaski (3, 4) → następny task (2).

**Z async/await** — `await` dzieli funkcję: reszta po `await` to kontynuacja w mikrotasku:

```js
async function f() {
  console.log('A');
  await null;              // wszystko poniżej = mikrotask
  console.log('B');
}
f();
console.log('C');
// A, C, B
```

**Zagnieżdżenie (poziom rozmowy senior):**

```js
setTimeout(() => {                       // task 1
  console.log('t1');
  Promise.resolve().then(() => console.log('p-in-t1'));   // mikrotask po t1, PRZED t2
});
setTimeout(() => console.log('t2'));     // task 2
// t1, p-in-t1, t2  — mikrotaski opróżniane po KAŻDYM tasku
```

## Timery — mity i fakty

- `setTimeout(fn, 0)` ≠ natychmiast: minimalne opóźnienie, kolejka, oraz **clamping do ~4 ms** przy zagnieżdżeniu ≥5 poziomów; w nieaktywnej karcie throttling do ≥1 s.
- `setTimeout` gwarantuje „nie wcześniej niż", nigdy „dokładnie po".
- `setInterval` może zlewać wykonania przy zajętym wątku; często lepszy rekurencyjny `setTimeout`.
- Do animacji: `requestAnimationFrame` (przed paintem, zsynchronizowany z odświeżaniem — [przydatne API](../05-dom-i-web-api/07-przydatne-api-przegladarki.md)).

## Długie taski i dzielenie pracy

Task >50 ms blokuje interakcje (metryka INP — [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md)). Strategie:

```js
await new Promise(r => setTimeout(r));   // oddaj wątek (yield) między porcjami
await scheduler.yield();                  // nowocześnie (Scheduler API)
```

- Dzielenie pętli na porcje (chunking), `requestIdleCallback` dla pracy niekrytycznej.
- Ciężkie obliczenia → [Web Worker](../05-dom-i-web-api/06-web-workers.md) (prawdziwy drugi wątek).
- Debounce/throttle dla gęstych zdarzeń — [optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md).

## Node.js — różnice (świadomość)

Node ma fazy event loopa (timers → I/O → check…), `setImmediate` i `process.nextTick` (priorytet ponad mikrotaskami). Szczegóły rzadziej pytane u frontendowca — kojarz, że model jest pokrewny, ale nie identyczny.

## Pułapki i częste błędy

- „setTimeout 0 wykona się od razu po bieżącej linijce" — nie: po WSZYSTKICH mikrotaskach i ew. wcześniejszych taskach.
- Synchoniczna ciężka pętla „z await w środku ale bez oddania wątku" — `await` na wartości nie-Promise i tak robi mikrotask, ale **nie oddaje** do renderingu między iteracjami tak, jak setTimeout (mikrotaski nie przepuszczają paintu!).
- Nieskończone dokładanie mikrotasków = zamrożona strona (gorsze niż setInterval).
- Zakładanie kolejności eventów input/timerów między przeglądarkami — trzymaj się gwarancji spec (mikrotaski), nie obserwacji.

## Pytania rekrutacyjne

1. **Opisz działanie event loopa.** — stack, Web APIs, task vs microtask, rendering między taskami.
2. **Co wypisze…?** — przykłady wyżej; ćwicz rozpiskę na głos.
3. **Czym różni się microtask od macrotaska? Podaj źródła obu.** — priorytet i moment opróżniania; then/queueMicrotask/MutationObserver vs setTimeout/eventy.
4. **Dlaczego `setTimeout(fn, 0)` nie wykonuje się natychmiast?** — kolejka, mikrotaski przed nim, clamping, throttling kart.
5. **Jak nie zablokować UI przy przetwarzaniu 100k rekordów?** — chunking + yield / worker; powiąż z INP.
6. **Dlaczego promise'y „wyprzedzają" setTimeout?** — mikrotaski opróżniane w całości po bieżącym tasku.

## Dalsza lektura

- [Jake Archibald: In the loop (talk)](https://www.youtube.com/watch?v=cCOL7MC4Pl0) — najlepsze wyjaśnienie wideo
- [Jake Archibald: Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
- [MDN: Event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Execution_model)
