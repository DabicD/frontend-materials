# Routing po stronie klienta

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [History API](../05-dom-i-web-api/07-przydatne-api-przegladarki.md), [Model komponentowy](./02-model-komponentowy.md)

W MPA nawigacja = nowy dokument z serwera. W [SPA](../08-architektura-aplikacji/02-spa-mpa-pwa.md) nawigacja = **podmiana widoku bez przeładowania**: router przechwytuje kliknięcia, zmienia URL przez History API i renderuje właściwy komponent. Wygląda prosto — diabeł tkwi w rzeczach, które przeglądarka robiła za darmo, a teraz musisz robić Ty.

## Mechanika (co robi każdy router pod spodem)

```js
// 1. Przechwyć kliknięcia w linki wewnętrzne (delegacja)
document.addEventListener('click', (e) => {
  const a = e.target.closest('a[href^="/"]');
  if (!a || e.metaKey || e.ctrlKey) return;      // nie psuj "otwórz w nowej karcie"!
  e.preventDefault();
  navigate(a.getAttribute('href'));
});

// 2. Zmień URL bez przeładowania + wyrenderuj
function navigate(path) {
  history.pushState({}, '', path);
  render(matchRoute(path));
}

// 3. Obsłuż wstecz/dalej
window.addEventListener('popstate', () => render(matchRoute(location.pathname)));
```

- **Dopasowanie tras:** wzorce `/products/:id` → parametry; trasy zagnieżdżone (layout + outlet); wildcard/404; rankowanie specyficzności. Standard platformy: `URLPattern`.
- **Hash routing** (`/#/path`) — historyczny fallback bez wsparcia serwera; dziś głównie legacy.
- **Wymóg serwera:** każda ścieżka musi zwrócić `index.html` (catch-all rewrite) — inaczej odświeżenie na `/products/42` daje 404 z serwera ([hosting](../16-devops-dla-frontendu/02-hosting-i-deployment.md)).

## Co przeglądarka robiła za darmo, a SPA musi odtworzyć

1. **Scroll restoration** — na górę przy nowej trasie, przywrócenie pozycji przy back/forward (`history.scrollRestoration = 'manual'` + własna logika).
2. **Ogłoszenie nawigacji czytnikom ekranu** — SPA nie przeładowuje strony, więc czytnik milczy; wzorce: live region z tytułem trasy, zarządzanie focusem ([a11y w praktyce](../10-dostepnosc/03-a11y-w-praktyce.md)).
3. **Tytuł i metadane** — `document.title` i meta per trasa ([SEO](../02-html/05-seo-i-metadane.md)).
4. **Stany ładowania** — dokument nie miga, ale dane lecą: skeletony, wskaźnik postępu, anulowanie poprzednich requestów ([AbortController](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).
5. **Prawdziwe linki** — zawsze `<a href>` (SEO, middle-click, dostępność), nie `div onClick` ([semantyka](../02-html/02-semantyka.md)).

Nowa **Navigation API** i cross-document **View Transitions** ([nowoczesny CSS](../03-css/08-nowoczesny-css.md)) zbliżają MPA do płynności SPA i upraszczają routery.

## Wzorce routingu aplikacyjnego

- **Route = jednostka code splittingu:** lazy loading komponentu trasy przez [dynamiczny import](../04-javascript/10-moduly.md) — bundle dzielony per widok ([optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md)); prefetch trasy przy hover/viewport na linku.
- **Guards / middleware nawigacji:** auth (redirect na login z `?returnTo=`), role, niedokończone formularze („masz niezapisane zmiany"). Pamiętaj: **guard kliencki to UX, nie bezpieczeństwo** — autoryzacja i tak na serwerze ([uwierzytelnianie](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)).
- **Ładowanie danych per trasa:** loader przed renderem (mniej spinnerów, dane gotowe z nawigacją) vs fetch-w-komponencie (prostsze, waterfall) — współczesne routery idą w loadery + cache ([wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)).
- **Stan w URL:** filtry, sortowanie, paginacja, otwarte modale → query params ([zarządzanie stanem](./04-zarzadzanie-stanem.md)); URL jako serializowany stan widoku.
- **Trasy zagnieżdżone:** layouty współdzielone (nav/sidebar) renderują dzieci w outlecie — mapowanie hierarchii UI na hierarchię URL.

## Pułapki i częste błędy

- Przechwytywanie kliknięć z modyfikatorami (Ctrl/Cmd/środkowy przycisk) — użytkownicy tracą „otwórz w nowej karcie".
- Brak catch-all na serwerze — 404 po odświeżeniu głębokiego linku.
- Utrata scrolla/focusa przy nawigacji — dezorientacja, szczególnie z czytnikiem ekranu.
- Race condition: wolniejsza odpowiedź starej trasy nadpisuje nową — anuluj/ignoruj nieaktualne.
- Stan widoku tylko w pamięci — filtry znikają po F5; nie da się podlinkować.
- Redirecty auth w renderze zamiast w guardzie — miganie chronionej treści.

## Pytania rekrutacyjne

1. **Jak działa routing SPA pod spodem?** — przechwycenie kliknięć, pushState, popstate, dopasowanie tras, render.
2. **Czemu odświeżenie strony SPA na podstronie może dać 404 i jak to naprawić?** — serwer nie zna tras klienta; rewrite do index.html.
3. **Co trzeba odtworzyć w SPA, co MPA ma za darmo?** — scroll, fokus/ogłoszenia a11y, tytuły, stany ładowania.
4. **Jak połączysz routing z code splittingiem?** — lazy route + prefetch.
5. **Guard tras a bezpieczeństwo?** — klient = UX; egzekwowanie na serwerze/API.

## Dalsza lektura

- [MDN: History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)
- [MDN: Navigation API](https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API)
- Dokumentacje routerów (React Router / Vue Router / TanStack Router) — czytane pod kątem wspólnych konceptów
