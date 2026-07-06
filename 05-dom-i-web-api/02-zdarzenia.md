# Zdarzenia (events)

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [DOM — manipulacja](./01-dom-manipulacja.md), [this i kontekst](../04-javascript/04-this-i-kontekst.md)

Zdarzenia to mechanizm reagowania na wszystko, co dzieje się na stronie: interakcje, sieć, media, cykl życia. Kluczem jest **model propagacji** (capturing → target → bubbling) — z niego wynika event delegation, `stopPropagation` i połowa pytań rekrutacyjnych.

## Rejestrowanie listenerów

```js
button.addEventListener('click', handleClick);
button.addEventListener('click', handleClick, { 
  once: true,       // usuń po pierwszym wywołaniu
  capture: false,   // faza (niżej)
  passive: true,    // "nie zawołam preventDefault" — patrz scroll
  signal: controller.signal,   // zbiorcze usuwanie przez AbortController
});
button.removeEventListener('click', handleClick);   // MUSI być ta sama referencja!
```

- `removeEventListener` z inną (nową) funkcją strzałkową nie usunie niczego — trzymaj referencję albo użyj `signal`:

```js
const ac = new AbortController();
window.addEventListener('resize', onResize, { signal: ac.signal });
// cleanup jednym ruchem (koniec życia komponentu):
ac.abort();
```

- Atrybuty `onclick="…"` w HTML i property `el.onclick` — legacy (jeden handler, mieszanie warstw); używaj `addEventListener`.
- Niesprzątane listenery = [memory leaki](../04-javascript/12-zaawansowane.md).

## Propagacja: capturing → target → bubbling

Kliknięcie w `<button>` wewnątrz `<div>` w `<body>`:

```
1. CAPTURING:  window → document → body → div        (w dół; listenery z {capture: true})
2. TARGET:     button                                 (element docelowy)
3. BUBBLING:   div → body → document → window         (w górę; domyślna faza listenerów)
```

- **`event.target`** — element, w który faktycznie kliknięto (najgłębszy).
- **`event.currentTarget`** — element, na którym wisi TEN listener (= `this` w zwykłej funkcji).
- **`event.stopPropagation()`** — zatrzymuje dalszą wędrówkę (ale inne listenery na bieżącym elemencie się wykonają; `stopImmediatePropagation` blokuje i je).
- Nie wszystkie zdarzenia bąbelkują: `focus`/`blur` nie (bąbelkujące odpowiedniki: `focusin`/`focusout`), `mouseenter`/`mouseleave` nie (bąbelkują `mouseover`/`mouseout`).

## Event delegation — wzorzec obowiązkowy

Zamiast listenera na każdym z 1000 wierszy — **jeden na kontenerze**, wykorzystujący bubbling:

```js
list.addEventListener('click', (e) => {
  const item = e.target.closest('[data-id]');   // od faktycznego celu w górę
  if (!item || !list.contains(item)) return;
  select(item.dataset.id);
});
```

Zalety: jeden listener (pamięć), **działa dla elementów dodanych później**, mniej sprzątania. Tak działają frameworki (React ma delegację wbudowaną). Ograniczenie: zdarzenia niebąbelkujące wymagają fazy capture.

## `preventDefault` i zdarzenia pasywne

- **`event.preventDefault()`** — blokuje akcję domyślną (nawigację linku, submit, pisanie w input, menu kontekstowe). Nie zatrzymuje propagacji (to `stopPropagation` — dwie różne rzeczy!).
- `event.cancelable` — czy default da się odwołać.
- **`{ passive: true }`** — obietnica „nie zawołam preventDefault": przeglądarka nie musi czekać z przewinięciem na wynik handlera → płynny scroll. Dla `touchstart`/`touchmove`/`wheel` na window/document jest to często domyślne. Ważny szczegół wydajności mobile.

## Ważniejsze zdarzenia w praktyce

- **Formularze:** `submit` (na form!), `input` (każda zmiana wartości), `change` (po zatwierdzeniu/utracie focusa), `focusin/focusout` — [formularze](../02-html/03-formularze.md).
- **Klawiatura:** `keydown` (z `e.key`: `'Enter'`, `'Escape'`, `'ArrowDown'`), unikaj przestarzałego `keyCode`.
- **Wskaźnik:** `click`, `dblclick`, `pointerdown/move/up` (**Pointer Events** unifikują mysz/dotyk/rysik — preferuj nad mouse*/touch*), `contextmenu`.
- **Scroll/rozmiar:** `scroll`, `resize` — gęste; debounce/throttle lub [obserwery](./05-observery.md).
- **Dokument/okno:** `DOMContentLoaded`, `load`, `visibilitychange`, `beforeunload`, `online/offline`, `storage` ([przechowywanie](./04-przechowywanie-danych.md)), `popstate`/hashchange ([routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)).

## Custom events — komunikacja komponentów

```js
el.dispatchEvent(new CustomEvent('cart:add', {
  detail: { productId: 42 },
  bubbles: true,               // bez tego nie wyjdzie ponad element!
  composed: true,              // przekracza granice Shadow DOM
}));
document.addEventListener('cart:add', e => e.detail.productId);
```

Naturalny sposób komunikacji „dziecko → świat" w Web Components i kodzie vanilla; we frameworkach zwykle zastąpione propsami/emitami ([model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)).

## Pułapki i częste błędy

- `removeEventListener` z nową referencją funkcji (bind/arrow tworzy nową!).
- `stopPropagation` „na wszelki wypadek" — psuje delegację, analitykę, zamykanie dropdownów; używaj tylko świadomie.
- Mylenie `target` z `currentTarget` przy delegacji.
- Ciężka praca w `scroll`/`resize`/`pointermove` bez throttle — jank ([optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)).
- `click` na dotyku vs `pointerup` — duchowe 300ms to przeszłość, ale kolejność zdarzeń touch→mouse→click wciąż zaskakuje.
- Listener na `document` dodawany w komponencie bez sprzątania.

## Pytania rekrutacyjne

1. **Opisz fazy propagacji zdarzenia.** — capturing/target/bubbling; jak podpiąć się na capture.
2. **Czym różni się `target` od `currentTarget`?** — faktyczny cel vs element z listenerem.
3. **Jak działa event delegation i po co?** — jeden listener + closest; wydajność, dynamiczne elementy.
4. **`preventDefault` vs `stopPropagation`?** — akcja domyślna vs wędrówka zdarzenia; niezależne.
5. **Co daje `{ passive: true }`?** — deklaracja braku preventDefault → scroll bez czekania na JS.
6. **Jak posprzątać wiele listenerów naraz?** — AbortController + signal.

## Dalsza lektura

- [javascript.info: Introduction to browser events](https://javascript.info/events)
- [MDN: Event reference](https://developer.mozilla.org/en-US/docs/Web/Events)
- [web.dev: Passive event listeners](https://web.dev/articles/uses-passive-event-listeners)
