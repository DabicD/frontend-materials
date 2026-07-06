# DOM — model i manipulacja

> **Poziom:** 🟢 podstawowy → 🟡 średni
> **Wymagana wiedza:** [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md), [Kolekcje i iteracja](../04-javascript/06-kolekcje-i-iteracja.md)

DOM (Document Object Model) to obiektowa reprezentacja dokumentu — drzewo węzłów, które JS może czytać i modyfikować. Frameworki ukrywają te operacje, ale pod spodem wykonują dokładnie je — znajomość DOM API to warunek rozumienia frameworków, debugowania i pisania testów.

## Drzewo węzłów

- Typy węzłów: **Element** (`<div>`), **Text** (także białe znaki między tagami!), **Comment**, **Document**.
- Nawigacja po elementach: `parentElement`, `children` (elementy), `firstElementChild`, `nextElementSibling`.
- Odpowiedniki `childNodes`/`firstChild` zwracają **wszystkie węzły** (z tekstowymi) — źródło niespodzianek.

## Wyszukiwanie elementów

```js
document.querySelector('.card a[href^="https"]');   // pierwszy pasujący (CSS selector)
document.querySelectorAll('.item');                  // NodeList (statyczna!)
element.querySelector('.child');                     // szukanie w poddrzewie
element.closest('.card');                            // najbliższy przodek pasujący (w górę!)
element.matches('.active');                          // czy element pasuje do selektora
document.getElementById('app');                      // najszybsze dla id
```

- `querySelectorAll` zwraca **statyczny** `NodeList` (snapshot); `getElementsByClassName/TagName` — **żywy** `HTMLCollection` (aktualizuje się sam — pętla po nim podczas modyfikacji DOM to pułapka).
- Obie kolekcje są **array-like** — do metod tablicowych: `[...nodeList]` / `Array.from` ([kolekcje](../04-javascript/06-kolekcje-i-iteracja.md)).
- `closest` + `matches` to fundament **event delegation** ([zdarzenia](./02-zdarzenia.md)).

## Tworzenie i wstawianie

```js
const li = document.createElement('li');
li.textContent = user.name;
li.className = 'item';
li.dataset.id = user.id;              // data-id="…"

list.append(li);                       // na koniec (może wiele naraz, także stringi)
list.prepend(li);                      // na początek
ref.before(li); ref.after(li);         // rodzeństwo
ref.replaceWith(li);
li.remove();

el.insertAdjacentHTML('beforeend', '<li>…</li>');   // parsowanie HTML w miejscu
const frag = document.createDocumentFragment();       // buforowanie wielu wstawek
```

- Element może być tylko w jednym miejscu — `append` istniejącego **przenosi** go.
- Masowe wstawianie: buduj w `DocumentFragment`/w oderwanym rodzicu i wstaw raz — mniej reflow ([optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)).
- `<template>` — deklaratywny szablon HTML: `template.content.cloneNode(true)`.

## Modyfikacja treści i atrybutów

```js
el.textContent = userInput;      // BEZPIECZNE — tekst, nie HTML
el.innerHTML = '<b>zaufany</b>'; // parsuje HTML — NIGDY z danymi użytkownika (XSS!)
el.insertAdjacentText(...);      // tekst w pozycji

el.setAttribute('aria-expanded', 'true'); el.getAttribute('href'); el.removeAttribute('hidden');
el.id, el.href, el.value, el.checked, el.disabled;   // właściwości (property)
el.classList.add/remove/toggle/contains('active');
el.style.setProperty('--x', '10px');                  // style inline (unikaj do layoutu)
el.dataset.userId;                                     // data-user-id
```

**Atrybut vs właściwość (klasyka rozmów):** atrybut = stan w HTML (początkowy), property = bieżący stan obiektu. `input.getAttribute('value')` to wartość początkowa; `input.value` — aktualnie wpisana. Dla booleanów liczy się **obecność** atrybutu, a property jest true/false.

**Bezpieczeństwo:** wstawianie nieoczyszczonego HTML (`innerHTML`, `insertAdjacentHTML`) z danych użytkownika = **XSS**. Tekst → `textContent`; HTML → sanityzacja (DOMPurify)/Sanitizer API — dom tematu: [ataki na frontend](../11-bezpieczenstwo/01-ataki-na-frontend.md).

## Geometria i scroll

```js
el.getBoundingClientRect();          // { top, left, width… } względem VIEWPORTU
el.offsetWidth, el.clientWidth;      // z borderem / bez (z paddingiem)
el.scrollTop, el.scrollHeight;
el.scrollIntoView({ behavior: 'smooth', block: 'start' });
window.scrollTo({ top: 0 });
```

Odczyty geometrii wymuszają synchroniczny layout — nie przeplataj z zapisami ([layout thrashing](../09-wydajnosc/03-optymalizacja-dzialania.md)). Do reagowania na widoczność/rozmiar: [obserwery](./05-observery.md), nie pętle odczytów.

## Pułapki i częste błędy

- `innerHTML +=` w pętli — reparsowanie całości za każdym razem + gubienie listenerów i stanu (focus, wartości inputów).
- Iterowanie po żywym `HTMLCollection` podczas usuwania elementów — przeskakiwanie co drugiego.
- Mylenie `textContent` (wszystko, szybkie) z `innerText` (uwzględnia CSS, **wymusza layout**).
- Selektory sprzężone ze strukturą (`div > div:nth-child(2) span`) — kruche; używaj klas/data-atrybutów (`[data-testid]` w testach — [testy komponentów](../13-testowanie/03-testy-komponentow-i-integracyjne.md)).
- Ręczna manipulacja DOM w aplikacji frameworkowej — walka z wirtualnym DOM/reaktywnością ([modele reaktywności](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md)).

## Pytania rekrutacyjne

1. **Różnica `querySelectorAll` vs `getElementsByClassName`?** — statyczny NodeList vs żywy HTMLCollection; konsekwencje przy modyfikacji.
2. **Atrybut vs property na przykładzie `input.value`?** — stan początkowy vs bieżący.
3. **Dlaczego `innerHTML` z danymi użytkownika jest groźny i co zamiast?** — XSS; textContent/sanityzacja.
4. **Jak wydajnie wstawić 1000 elementów?** — fragment/batch, jeden append; jeszcze lepiej: wirtualizacja listy.
5. **Co robi `closest()` i gdzie się przydaje?** — szukanie przodka; event delegation.

## Dalsza lektura

- [MDN: Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [javascript.info: Document](https://javascript.info/document)
