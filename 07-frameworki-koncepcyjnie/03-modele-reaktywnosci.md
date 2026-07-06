# Modele reaktywności: Virtual DOM, signals i spółka

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Model komponentowy](./02-model-komponentowy.md), [Proxy](../04-javascript/12-zaawansowane.md), [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)

Reaktywność = **jak framework dowiaduje się, że stan się zmienił, i jak ustala, co zaktualizować w DOM**. To główna oś różnic między frameworkami i temat, po którym poznaje się seniora: nie „czy umiesz hooka", tylko „czy wiesz, co się dzieje po `setState`".

## Problem

DOM jest wolny nie sam z siebie — drogie są [reflow/repaint](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md) i praca na dużych drzewach. Framework chce: przyjąć nowy stan → dotknąć **minimalnej liczby** węzłów DOM. Strategie różnią się tym, **co śledzą** i **kiedy liczą różnice**.

## Strategia 1: Virtual DOM (React, Vue 2/3-vdom, Preact)

Model: **re-render + diff**. Komponent to funkcja zwracająca lekki opis UI (drzewo obiektów JS — VDOM). Po zmianie stanu:

1. Wywołaj funkcję ponownie → nowe drzewo VDOM.
2. **Diff (reconciliation):** porównaj nowe drzewo ze starym.
3. Zaaplikuj do prawdziwego DOM tylko różnice (patch).

```
stan → f(stan) → VDOM' ──diff z VDOM──▶ minimalne operacje DOM
```

- Heurystyki diffu: porównanie po typie elementu; listy po **kluczach** (`key`) — stąd żelazna zasada: klucz stabilny i unikalny, **nie index** (index przy wstawieniu w środek listy powoduje przemapowanie stanu do złych elementów).
- Konsekwencja modelu: **domyślnie re-renderuje się całe poddrzewo** właściciela stanu — optymalizacje (memo, selektory) są ręczne; „gruboziarnista" reaktywność.
- Zaleta: prostota modelu mentalnego (UI = czysta funkcja stanu, zawsze przeliczana), łatwe testowanie, elastyczność (render dowolnej struktury).
- React dokłada: batching aktualizacji, priorytety (concurrent), a kompilator (React Compiler) automatyzuje memoizację.

## Strategia 2: Fine-grained reactivity / signals (Solid, Vue reactivity core, Svelte 5 runes, Angular signals)

Model: **graf zależności**. Stan to **signal** (obserwowalna wartość); odczyt signala w trakcie obliczeń **rejestruje zależność**; zapis — powiadamia dokładnie tych, którzy zależą:

```js
const [count, setCount] = createSignal(0);
const double = createMemo(() => count() * 2);     // zależy od count
createEffect(() => (el.textContent = double()));   // zależy od double
setCount(5);   // → przelicza double → aktualizuje TYLKO ten węzeł tekstowy
```

- **Komponent wykonuje się RAZ** (setup); potem żyją tylko subskrypcje → aktualizacje punktowe, bez diffowania drzew.
- Implementacje śledzenia: funkcje-gettery (Solid), **Proxy** (Vue `reactive` — [mechanika](../04-javascript/12-zaawansowane.md)), kompilator znajdujący zależności (Svelte).
- Pojęcia uniwersalne: **signal** (wartość), **computed/memo** (wartość pochodna, cache'owana), **effect** (reakcja na zmiany: DOM, I/O).
- Pułapka modelu: **utrata reaktywności** przez destrukturyzację/skopiowanie wartości poza śledzeniem (czytasz raz, nie subskrybujesz) — odpowiednik „stale closure".
- Zaleta: wydajność bez ręcznej memoizacji; koszt: model „co jest śledzone, a co nie" trzeba rozumieć.

## Strategia 3: kompilacja i inne

- **Svelte (≤4):** kompilator zamienia przypisania `count += 1` w kod aktualizujący DOM — reaktywność „wpisana" w build (Svelte 5 przeszedł na runes/signals).
- **Angular (klasycznie):** zone.js + dirty checking — po każdym zdarzeniu sprawdź powiązania; kierunek rozwoju: signals.
- Wspólny trend ekosystemu: **od diffowania w runtime do sygnałów i kompilacji** — mniej pracy w przeglądarce.

## Uniwersalne konsekwencje praktyczne (niezależne od frameworka)

1. **Niemutowalność vs śledzenie mutacji:** VDOM-owy świat porównuje **referencje** — mutacja `arr.push()` bywa niewidoczna, twórz nowe obiekty ([spread](../04-javascript/07-skladnia-es6-plus.md), [kolekcje](../04-javascript/06-kolekcje-i-iteracja.md)). Świat Proxy odwrotnie — mutacje są śledzone, ale **podmiana całego obiektu** poza proxy gubi reaktywność. Musisz wiedzieć, w którym świecie jesteś.
2. **Batching:** frameworki grupują aktualizacje (mikrotask/kolejka — [event loop](../04-javascript/09-event-loop.md)); odczyt DOM „zaraz po" zmianie stanu wymaga dedykowanego haka (`nextTick`/effect po fladze), nie `setTimeout` na oko.
3. **Wartości pochodne licz, nie przechowuj:** duplikowanie stanu pochodnego (np. `filteredItems` w stanie obok `items` i `filter`) = desynchronizacja; używaj computed/memo.
4. **Klucze list** — wszędzie ta sama zasada i te same bugi.
5. **Głębokie drzewa stanu:** selektory/granulacja subskrypcji, by zmiana jednego pola nie odświeżała świata ([zarządzanie stanem](./04-zarzadzanie-stanem.md)).

## Pułapki i częste błędy

- Mutacja stanu w świecie porównań referencyjnych („czemu się nie odświeża?").
- `key={index}` na dynamicznych listach.
- Destrukturyzacja reaktywnego źródła w świecie signals/Proxy — martwe wartości.
- Przechowywanie wartości pochodnych zamiast liczenia.
- Efekty łańcuchowe (effect ustawia stan, który triggeruje effect…) — pętle i lawiny renderów; projektuj przepływ jednokierunkowo.
- Przedwczesna memoizacja wszystkiego w VDOM — koszt złożoności bez pomiaru ([profilowanie](../14-narzedzia/05-devtools.md)).

## Pytania rekrutacyjne

1. **Jak działa Virtual DOM i po co, skoro DOM można zmieniać wprost?** — re-render+diff jako model deklaratywny z akceptowalnym kosztem; minimalizacja operacji na prawdziwym DOM.
2. **Czym są signals i czym różnią się od VDOM?** — graf zależności, aktualizacje punktowe, komponent wykonany raz vs re-render poddrzewa.
3. **Dlaczego `key` w listach nie powinien być indexem?** — tożsamość elementów przy diffie; przemapowanie stanu.
4. **Dlaczego frameworki wymagają niemutowalnych aktualizacji (lub odwrotnie — śledzą mutacje)?** — porównania referencji vs Proxy tracking.
5. **Co to jest batching aktualizacji i jaki problem rozwiązuje?** — jeden render na wiele zmian; spójność i wydajność.

## Dalsza lektura

- [React docs: Render and Commit + Preserving and Resetting State](https://react.dev/learn/render-and-commit)
- [Solid docs: Fine-grained reactivity](https://docs.solidjs.com/advanced-concepts/fine-grained-reactivity)
- [Vue docs: Reactivity in Depth](https://vuejs.org/guide/extras/reactivity-in-depth.html)
- Ryan Carniato — artykuły/talki o reaktywności (twórca Solid)
