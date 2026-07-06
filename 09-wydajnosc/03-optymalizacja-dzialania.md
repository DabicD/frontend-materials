# Optymalizacja działania (runtime performance)

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Event loop](../04-javascript/09-event-loop.md), [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md), [Core Web Vitals](./01-core-web-vitals.md)

Strona załadowana ≠ strona szybka: scroll się zacina, wpisywanie laguje, klik reaguje po sekundzie. Runtime performance = utrzymanie klatek (16,7 ms przy 60 Hz) i szybkiej reakcji na input (INP). Dwa wrogowie: **długie taski JS** i **niepotrzebna praca renderowania**.

## Długie taski i dzielenie pracy

Main thread robi wszystko ([event loop](../04-javascript/09-event-loop.md)); task >50 ms opóźnia każdą interakcję.

```js
// zamiast: przetworzenie 50k rekordów w jednej pętli (główny wątek stoi)
for (const chunk of chunks(records, 500)) {
  process(chunk);
  await scheduler.yield();           // oddaj wątek: input/paint mają szansę
}
```

- Chunking + `scheduler.yield()` / `setTimeout(0)`; praca niekrytyczna → `requestIdleCallback`.
- Ciężkie obliczenia → [Web Worker](../05-dom-i-web-api/06-web-workers.md).
- Diagnoza: Performance panel → czerwone „Long Tasks" ([DevTools](../14-narzedzia/05-devtools.md)).

## Debounce i throttle (dom tematu)

Gęste zdarzenia (`input`, `scroll`, `resize`, `pointermove`) nie mogą wykonywać drogiej pracy za każdym strzałem:

```js
// DEBOUNCE — wykonaj PO USTANIU serii (wyszukiwarka: czekaj aż przestanie pisać)
function debounce(fn, ms) {
  let t;
  return (...args) => { clearTimeout(t); t = setTimeout(() => fn(...args), ms); };
}

// THROTTLE — wykonuj CO NAJWYŻEJ raz na N ms (scroll: aktualizuj, ale z limitem)
function throttle(fn, ms) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= ms) { last = now; fn(...args); }
  };
}
```

- **Debounce** = „poczekaj na koniec" (autosave, walidacja, autocomplete — tu też [AbortController](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md) dla starych requestów).
- **Throttle** = „regularnie, ale rzadziej" (pozycja scrolla, drag, resize — często lepszy rAF-throttle: wykonuj raz na klatkę).
- Klasyczne pytanie: napisz oba z ręki i wyjaśnij różnicę.
- Zanim throttlujesz scroll — sprawdź, czy problemu nie rozwiązuje [IntersectionObserver](../05-dom-i-web-api/05-observery.md)/CSS.

## Praca z DOM i layout thrashing

Z [pipeline'u](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md): odczyt geometrii przy „brudnych" stylach wymusza **synchroniczny layout**. Przeplatanie w pętli = thrashing:

```js
// ŹLE: odczyt→zapis→odczyt→zapis — layout per iteracja
items.forEach(el => { el.style.height = el.offsetHeight * 2 + 'px'; });

// DOBRZE: najpierw WSZYSTKIE odczyty, potem WSZYSTKIE zapisy
const heights = items.map(el => el.offsetHeight);
items.forEach((el, i) => { el.style.height = heights[i] * 2 + 'px'; });
```

- Batchuj modyfikacje (fragment, wymiana klas zamiast wielu styli — [manipulacja DOM](../05-dom-i-web-api/01-dom-manipulacja.md)).
- Animuj compositor-friendly (`transform`/`opacity` — [animacje](../03-css/07-animacje-i-transformacje.md)).
- **`content-visibility: auto`** + `contain-intrinsic-size` — przeglądarka pomija render poza ekranem (długie strony); `contain` — izolacja layoutu widżetu.

## Długie listy: wirtualizacja

10 000 wierszy w DOM = wolny layout, hit pamięci, mulący scroll — niezależnie od frameworka. **Wirtualizacja:** renderuj tylko widoczne ~30 + bufor, pozycjonuj transformem w kontenerze o pełnej wysokości; przy scrollu podmieniaj zawartość (biblioteki: TanStack Virtual, react-window itd.).

- Konsekwencje: Ctrl+F nie znajdzie niewyrenderowanych, [dostępność](../10-dostepnosc/02-aria-i-semantyka.md) wymaga uwagi (aria-rowcount), sticky/dynamiczne wysokości komplikują.
- Alternatywy zanim zwirtualizujesz: paginacja, infinite scroll z rozsądnym limitem, `content-visibility`.

## Warstwa frameworka

Framework dokłada własną pracę ([reaktywność](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md)): re-rendery poddrzew, diffowanie. Uniwersalnie:

- Profiluj narzędziami frameworka (React/Vue DevTools profiler) — **co** się renderuje i czemu.
- Zawężaj zasięg zmian: granulacja stanu/selektory ([zarządzanie stanem](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)), memoizacja **po pomiarze**, stabilne `key`.
- Nie trzymaj w stanie rzeczy zmieniających się co piksel (pozycja kursora) — ref/zmienna poza reaktywnością, aktualizacja w rAF.

## Pamięć

Wydajność degradującą się **z czasem** (po godzinie użycia) powodują leaki i rosnące struktury — diagnostyka i wzorce: [pamięć i GC](../04-javascript/12-zaawansowane.md).

## Pułapki i częste błędy

- Optymalizacja bez profilowania — „wydaje mi się, że to ta pętla" (zwykle nie).
- Debounce inputu użytkownika tam, gdzie oczekuje natychmiastowej reakcji UI (echo znaków!) — debounce dotyczy **kosztownych skutków**, nie samego echa.
- `scroll` listener liczący `getBoundingClientRect` na 50 elementach.
- Wirtualizacja 50-elementowej listy (złożoność bez zysku) / brak wirtualizacji przy 10k.
- Memoizacja wszystkiego „profilaktycznie" — koszt porównań i pamięci bez pomiaru.
- Testowanie na M3 Max — rzeczywistość to średni Android; CPU throttling 4–6×.

## Pytania rekrutacyjne

1. **Napisz debounce i throttle; różnice i zastosowania.** — klasyk live-codingu.
2. **Czym jest layout thrashing i jak go unikać?** — wymuszony synchroniczny layout; batch odczytów/zapisów.
3. **Jak wyrenderujesz listę 100k elementów?** — wirtualizacja: mechanika, kompromisy, alternatywy.
4. **Interakcja reaguje po 800 ms — jak szukasz przyczyny?** — Performance panel: long task? handler? layout? re-render frameworka? (rozbij INP na składowe).
5. **Jak nie blokować głównego wątku przy ciężkiej pracy?** — chunk+yield, idle, worker; kryteria wyboru.

## Dalsza lektura

- [web.dev: Optimize long tasks / INP](https://web.dev/articles/optimize-long-tasks)
- [Chrome DevTools: Analyze runtime performance](https://developer.chrome.com/docs/devtools/performance)
- [TanStack Virtual](https://tanstack.com/virtual/latest)
