# Po co są frameworki?

> **Poziom:** 🟢 podstawowy (koncepcyjnie), 🟡 dla pełnego docenienia
> **Wymagana wiedza:** [DOM — manipulacja](../05-dom-i-web-api/01-dom-manipulacja.md), [Zdarzenia](../05-dom-i-web-api/02-zdarzenia.md)

Zanim ocenisz „React vs Vue", musisz umieć odpowiedzieć na pytanie głębsze: **jaki problem w ogóle rozwiązują frameworki?** Ta sekcja (07-*) jest celowo framework-agnostic: koncepty są wspólne, różni się składnia — kto rozumie koncepty, uczy się dowolnego frameworka w tygodnie, nie miesiące.

## Problem: synchronizacja stanu z widokiem

Aplikacja = **stan** (dane) + **widok** (DOM) + **interakcje** (zmieniają stan). W vanilla JS synchronizację robisz ręcznie:

```js
let items = [];
function addItem(item) {
  items.push(item);
  // …i teraz RĘCZNIE zaktualizuj każde miejsce w DOM, które zależy od items:
  listEl.append(renderItem(item));
  counterEl.textContent = items.length;
  emptyStateEl.hidden = items.length > 0;
  submitBtn.disabled = items.length === 0;
}
```

Przy 3 elementach UI — trywialne. Przy 50 widokach zależnych od 30 pól stanu — **kombinatoryczna eksplozja**: każda mutacja musi pamiętać o każdym zależnym fragmencie DOM. Zapomnisz jednego → UI „rozjeżdża się" ze stanem (klasa bugów „stale UI"). Do tego: sprzątanie listenerów, kolejność aktualizacji, wydajność.

## Rozwiązanie: UI jako funkcja stanu

Frameworki odwracają model: **deklarujesz, jak widok wynika ze stanu** (`UI = f(state)`), a framework pilnuje synchronizacji:

```jsx
// pseudokod deklaratywny — czytaj: "widok WYGLĄDA TAK dla danego stanu"
<list>{items.map(renderItem)}</list>
<counter>{items.length}</counter>
<empty hidden={items.length > 0} />
<submit disabled={items.length === 0} />
```

Zmieniasz stan → framework sam ustala, **co** w DOM trzeba dotknąć (strategie: [modele reaktywności](./03-modele-reaktywnosci.md)). Programujesz **deklaratywnie** („co ma być"), nie **imperatywnie** („jak to osiągnąć krok po kroku").

## Co jeszcze dają frameworki

1. **Model komponentowy** — kompozycja UI z izolowanych, reużywalnych klocków ([komponenty](./02-model-komponentowy.md)).
2. **Cykl życia** — uporządkowane haki na mount/update/unmount (sprzątanie subskrypcji! — [memory leaki](../04-javascript/12-zaawansowane.md)).
3. **Ekosystem** — routing ([routing SPA](./05-routing-po-stronie-klienta.md)), zarządzanie stanem ([stan](./04-zarzadzanie-stanem.md)), narzędzia dev, biblioteki komponentów, [SSR](../08-architektura-aplikacji/01-wzorce-renderowania.md).
4. **Konwencje zespołowe** — wspólny język i struktura; nowa osoba w projekcie wie, gdzie szukać.
5. **Bezpieczeństwo domyślne** — automatyczne escapowanie interpolacji (ochrona przed [XSS](../11-bezpieczenstwo/01-ataki-na-frontend.md) — dopóki nie użyjesz „innerHTML-owych" furtek).

## Koszty (o których senior mówi głośno)

- **Waga i wydajność startu:** runtime frameworka + Twój kod → JS do pobrania/parsowania/wykonania ([optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md)). Stąd nurty: kompilacja zamiast runtime (Svelte), resumability (Qwik), islands ([wzorce renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md)).
- **Abstrakcja przecieka:** gdy coś nie działa, debugujesz i framework, i platformę; bez znajomości DOM/eventów jesteś bezradny.
- **Churn i lock-in:** migracje wersji, przepisywanie; wiedza platformowa (HTML/CSS/JS/HTTP) starzeje się wolniej niż frameworkowa.
- **Overkill dla prostych stron:** landing z formularzem nie potrzebuje SPA — [MPA/SSG](../08-architektura-aplikacji/02-spa-mpa-pwa.md) bywa lepsze.

**Kiedy framework się broni:** UI z bogatym, współdzielonym stanem i częstymi aktualizacjami (dashboardy, edytory, e-commerce checkout, komunikatory). **Kiedy przemyśl:** treść statyczna, proste interakcje — wystarczy HTML+odrobina JS (albo islands).

## Pułapki i częste błędy (myślowe)

- „Uczę się Reacta zamiast JavaScriptu" — braki w [closures](../04-javascript/03-scope-hoisting-closures.md), [asynchroniczności](../04-javascript/08-asynchronicznosc.md), [event loopie](../04-javascript/09-event-loop.md) wracają jako „magiczne" bugi frameworkowe.
- Utożsamianie frameworka z architekturą — framework organizuje widok; architektura aplikacji ([struktura projektu](../08-architektura-aplikacji/04-struktura-projektu.md)) to Twoja odpowiedzialność.
- Dobór frameworka pod hype zamiast pod problem i zespół.
- Walka z frameworkiem (ręczne grzebanie w DOM obok niego — [manipulacja](../05-dom-i-web-api/01-dom-manipulacja.md)).

## Pytania rekrutacyjne

1. **Jaki problem rozwiązują frameworki frontendowe?** — synchronizacja stan↔widok; deklaratywność; komponenty.
2. **Deklaratywne vs imperatywne UI — wyjaśnij na przykładzie.** — „co" vs „jak"; konsekwencje dla utrzymania.
3. **Jakie są koszty użycia frameworka?** — bundle/hydracja, abstrakcja, churn; kiedy vanilla/MPA.
4. **Czy potrafiłbyś zbudować mini-framework? Co by zawierał?** — stan → render, komponenty, zdarzenia, diffing/sygnały (świetne pytanie sprawdzające zrozumienie).

## Dalsza lektura

- [Patterns.dev](https://www.patterns.dev/) — wzorce niezależne od frameworka
- [Frontend Masters: Framework-less / vanilla approaches](https://frontendmasters.com/)
- Tom Dale, „Rethinking Best Practices" (klasyczny kontekst historyczny)
