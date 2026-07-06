# Asynchroniczność: callbacki, Promises, async/await

> **Poziom:** 🟡 średni (dom tematu Promises — inne pliki tu linkują)
> **Wymagana wiedza:** [Funkcje](./02-funkcje.md), [Closures](./03-scope-hoisting-closures.md)

JavaScript wykonuje kod w **jednym wątku** — długie operacje (sieć, timery, dysk) nie mogą blokować. Zamiast tego są **asynchroniczne**: startują, a wynik wraca później przez callback lub Promise. *Jak* silnik to koordynuje, opisuje [event loop](./09-event-loop.md); tutaj — jak o asynchroniczności **pisać**.

## Callbacki i ich piekło

```js
getUser(id, (err, user) => {
  if (err) return handle(err);
  getOrders(user, (err, orders) => {      // zagnieżdżenie rośnie…
    …
  });
});
```

Problemy: **callback hell** (piramida zagłady), ręczna propagacja błędów na każdym poziomie, brak kompozycji. Odpowiedź języka: Promises.

## Promise — obiekt przyszłej wartości

Promise jest w jednym z 3 stanów: **pending** → **fulfilled** (z wartością) albo **rejected** (z powodem). Przejście jest **jednorazowe i nieodwracalne** (settled).

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => resolve(42), 1000);
});

p.then(v => v * 2)                 // .then ZWRACA NOWY Promise → łańcuchowanie
 .then(v => fetchSomething(v))     // zwrócenie Promise = czekanie na niego
 .catch(err => console.error(err)) // łapie odrzucenie z KAŻDEGO ogniwa wyżej
 .finally(() => hideSpinner());    // zawsze; nie dostaje wartości
```

**Zasady łańcucha (częste pytania):**
- Wartość zwrócona w `then` → wartość następnego ogniwa; zwrócony Promise → „rozpakowany" wynik; rzucony wyjątek → rejection.
- `.catch` obsługuje błąd i **wraca do toru fulfilled** (chyba że znów rzuci).
- Brak `.catch` = **unhandled rejection** (błąd w konsoli, event `unhandledrejection`).
- Callbacki `then` wykonują się jako **mikrotaski** — zawsze asynchronicznie, nawet dla rozstrzygniętego Promise ([event loop](./09-event-loop.md)).

## async/await — składnia nad Promises

```js
async function loadUser(id) {          // async ⇒ funkcja ZAWSZE zwraca Promise
  try {
    const res = await fetch(`/api/users/${id}`);   // await = czekaj na Promise
    if (!res.ok) throw new HttpError(res);
    return await res.json();
  } catch (err) {                       // łapie i rejections, i zwykłe wyjątki
    …                                    // strategie: patrz obsługa błędów
  } finally { … }
}
```

- `await` **nie blokuje wątku** — zawiesza funkcję, reszta programu działa.
- `try/catch` działa naturalnie; szczegóły wzorców: [obsługa błędów](./11-obsluga-bledow.md).
- Top-level `await` — dostępny w modułach ESM.

**Sekwencyjnie vs równolegle — pytanie nr 1 w code review:**

```js
// ŹLE (gdy niezależne): 300ms + 300ms = 600ms
const a = await fetchA();
const b = await fetchB();

// DOBRZE: max(300, 300) = 300ms
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

## Kombinatory Promise

| Kombinator | Rozstrzyga się, gdy… | Odrzuca, gdy… | Typowe użycie |
|---|---|---|---|
| `Promise.all` | wszystkie fulfilled → tablica wyników | **pierwszy** rejected (fail-fast) | równoległe zależne od kompletu |
| `Promise.allSettled` | wszystkie settled → `[{status, value/reason}]` | nigdy | raport zbiorczy, częściowe sukcesy |
| `Promise.race` | pierwszy **settled** (sukces LUB błąd) | jw. | timeouty |
| `Promise.any` | pierwszy **fulfilled** | wszystkie rejected (`AggregateError`) | najszybsze źródło/mirror |

Plus: `Promise.resolve(v)` / `Promise.reject(e)`, `Promise.withResolvers()` (wyciągnięte resolve/reject — czystszy „deferred").

**Timeout dla operacji (wzorzec):**

```js
const timeout = ms => new Promise((_, rej) => setTimeout(() => rej(new Error('Timeout')), ms));
const data = await Promise.race([fetchData(), timeout(5000)]);
// dla fetch właściwsze: AbortController — patrz plik o fetch
```

Anulowanie requestów i race conditions przy pobieraniu danych: [fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md) i [wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md).

## Pętle a await

```js
for (const id of ids) await process(id);              // sekwencyjnie — OK gdy zależne
await Promise.all(ids.map(id => process(id)));        // równolegle — zwykle to chcesz
ids.forEach(async id => await process(id));           // ✘ forEach NIE czeka — "fire and forget"
for await (const chunk of stream) { … }               // async iterables (strumienie)
```

Równoległość z limitem współbieżności (np. max 5 naraz) — pula workerów / biblioteka `p-limit`; umieć naszkicować na rozmowie.

## Pułapki i częste błędy

- Zgubiony `await`/`return` w środku łańcucha — „dziura", przez którą błędy uciekają, a kolejność się psuje.
- `forEach(async …)` — patrz wyżej.
- Sekwencyjne awaity niezależnych operacji.
- `await` w pętli po tablicy do renderu — sekwencyjny wodospad requestów.
- Miks stylów: `.then` wewnątrz `async` bez potrzeby — wybierz jeden styl (async/await), then zostaw do prostych łańcuchów.
- Konstruktor Promise wokół czegoś, co już zwraca Promise (antywzorzec „explicit construction").
- Nieobsłużone rejection w „fire and forget" (`void promise` + własny handler, albo po prostu await).

## Pytania rekrutacyjne

1. **Stany Promise i co je zmienia?** — pending/fulfilled/rejected; jednorazowość.
2. **Co zwraca `.then` i jak działa propagacja błędów w łańcuchu?** — nowy Promise; wyjątek/rejection spada do najbliższego catch; catch „naprawia" tor.
3. **`Promise.all` vs `allSettled` vs `race` vs `any`?** — tabela wyżej + przypadki użycia.
4. **Czy `await` blokuje wątek? Co się dzieje pod spodem?** — nie; zawieszenie funkcji, kontynuacja jako mikrotask ([event loop](./09-event-loop.md)).
5. **Jak wykonać 100 requestów z limitem 5 równoległych?** — kolejka/pula; szkic implementacji.
6. **Dlaczego `array.forEach(async …)` nie czeka?** — forEach ignoruje zwracane Promise; alternatywy: for...of / Promise.all(map).

## Dalsza lektura

- [javascript.info: Promises, async/await](https://javascript.info/async) — najlepszy darmowy kurs tego tematu
- [MDN: Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [MDN: Promise — kombinatory](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
