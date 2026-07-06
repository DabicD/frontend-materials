# Obserwery: Intersection, Resize, Mutation

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [DOM — manipulacja](./01-dom-manipulacja.md), [Zdarzenia](./02-zdarzenia.md)

Obserwery to nowoczesna odpowiedź na pytanie „jak zareagować na zmianę w DOM/układzie **bez** kosztownego odpytywania (polling) i bez zapychania zdarzeń scroll/resize". Działają asynchronicznie i wydajnie — przeglądarka batchuje powiadomienia.

## IntersectionObserver — widoczność w viewporcie

„Powiadom mnie, gdy element wejdzie/wyjdzie z widoku (lub kontenera przewijania)":

```js
const io = new IntersectionObserver((entries) => {
  for (const entry of entries) {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
      io.unobserve(entry.target);            // jednorazowo? przestań obserwować
    }
  }
}, {
  root: null,                 // null = viewport; można wskazać scrollowany kontener
  rootMargin: '200px 0px',    // "poszerz" obszar — reaguj z wyprzedzeniem
  threshold: [0, 0.5, 1],     // przy jakim % widoczności strzelać
});
io.observe(document.querySelector('.lazy-section'));
```

**Zastosowania (kanoniczne):**
- **Lazy loading** treści/komponentów poniżej fold (obrazy mają natywne `loading="lazy"` — [multimedia](../02-html/04-multimedia-i-grafika.md); IO do wszystkiego innego).
- **Infinite scroll** — sentinel na końcu listy ([system design: infinite scroll](../17-rekrutacja/04-frontend-system-design.md)).
- Animacje „reveal on scroll" (coraz częściej czysty CSS — [scroll-driven animations](../03-css/07-animacje-i-transformacje.md)).
- Analityka widoczności (reklamy, sekcje), auto-pauza wideo poza ekranem.
- Aktywny punkt spisu treści (scrollspy).

Zamiast listenera `scroll` z `getBoundingClientRect()` w pętli (layout w każdej klatce — [optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)) — IO liczy to poza main threadem.

## ResizeObserver — zmiana rozmiaru elementu

„Powiadom, gdy **element** (nie okno!) zmieni rozmiar" — reaguje na wszystko: zmianę treści, flexbox, sidebar, devtools:

```js
const ro = new ResizeObserver((entries) => {
  for (const { target, contentBoxSize } of entries) {
    const width = contentBoxSize[0].inlineSize;
    target.classList.toggle('compact', width < 400);
  }
});
ro.observe(cardElement);
```

Zastosowania: komponenty adaptujące się do kontenera (dziś zwykle lepiej: [container queries](../03-css/06-responsywnosc.md) — RO zostaje, gdy potrzebna **logika JS**: przeliczenie canvasa, wykresu, wirtualizowanej listy), synchronizacja wymiarów. Uwaga na pętlę: modyfikacja rozmiaru w callbacku → „ResizeObserver loop limit exceeded".

## MutationObserver — zmiany w drzewie DOM

„Powiadom, gdy ktoś zmieni DOM": dodane/usunięte węzły, atrybuty, tekst:

```js
const mo = new MutationObserver((mutations) => {
  for (const m of mutations) {
    if (m.type === 'childList') m.addedNodes.forEach(enhance);
  }
});
mo.observe(container, {
  childList: true, subtree: true,        // węzły w całym poddrzewie
  attributes: true, attributeFilter: ['class'],
  characterData: false,
});
mo.disconnect();   // sprzątanie!
```

Zastosowania: integracja z kodem, którego nie kontrolujesz (widgety third-party, CMS), biblioteki a11y reagujące na zmiany, śledzenie zmian przez rozszerzenia/testy. Callback działa jako **mikrotask** ([event loop](../04-javascript/09-event-loop.md)). We własnej aplikacji zwykle **nie jest potrzebny** — sam wiesz, kiedy zmieniasz DOM; MO bywa objawem walki z frameworkiem.

## Wspólne zasady

- Jeden obserwer może obserwować **wiele elementów** (`observe()` wielokrotnie) — twórz jeden, nie per element.
- **Sprzątaj:** `unobserve(el)` / `disconnect()` przy śmierci komponentu — [memory leaki](../04-javascript/12-zaawansowane.md).
- Callbacki dostają **batch** wpisów — iteruj po wszystkich, nie zakładaj pojedynczego.
- Pierwsze wywołanie IO/RO następuje od razu po `observe` (stan początkowy) — wygodne, ale bywa zaskoczeniem.

## Pułapki i częste błędy

- Scroll listener + getBoundingClientRect tam, gdzie należy użyć IO.
- Zapominanie `rootMargin` przy lazy loadingu — treść ładuje się dopiero „na styk", użytkownik widzi placeholdery.
- `threshold: 1` dla wysokich elementów — nigdy nie osiągną 100% widoczności w niskim viewporcie.
- Modyfikowanie DOM w callbacku MutationObserver bez warunków stopu — nieskończona pętla obserwacji.
- Brak disconnect w SPA — obserwery żyją po odmontowaniu widoku.

## Pytania rekrutacyjne

1. **Jak zaimplementujesz lazy loading sekcji strony?** — IO + rootMargin + unobserve; porównaj z natywnym loading="lazy".
2. **Czym różni się ResizeObserver od zdarzenia `resize`?** — element vs okno; reaguje na każdą przyczynę zmiany rozmiaru.
3. **Kiedy MutationObserver to właściwe narzędzie, a kiedy code smell?** — obcy DOM vs własny (wiesz, kiedy zmieniasz).
4. **Dlaczego IO jest wydajniejszy od scroll+gBCR?** — brak synchronicznego layoutu, praca poza main threadem, batching.
5. **Container queries vs ResizeObserver?** — styl w CSS vs logika w JS; CQ pierwsze, RO gdy trzeba liczyć.

## Dalsza lektura

- [MDN: IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [MDN: ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver)
- [MDN: MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)
