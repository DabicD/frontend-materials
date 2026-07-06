# Model komponentowy

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Po co frameworki](./01-po-co-frameworki.md), [Funkcje](../04-javascript/02-funkcje.md)

Komponent to jednostka kompozycji UI: **własny szablon + logika + (opcjonalnie) style + stan**, z jawnym kontraktem wejścia/wyjścia. Wszystkie współczesne frameworki — mimo różnic składni — realizują ten sam model. Rozumiesz model → czytasz każdy framework.

## Kontrakt komponentu: props in, events out

```
        props (dane W DÓŁ)
   ┌──────────▼──────────┐
   │      Komponent       │   własny stan lokalny
   └──────────▲──────────┘
        eventy/callbacki (sygnały W GÓRĘ)
```

- **Props** — wejście: dane i konfiguracja od rodzica. **Tylko do odczytu** — komponent nie modyfikuje swoich props (inaczej przepływ danych staje się nieprzewidywalny).
- **Eventy/callbacki** — wyjście: dziecko **informuje** („kliknięto", „zmieniono wartość"), rodzic **decyduje**, co z tym zrobić. React: callback w props (`onChange={fn}`); Vue/Angular/Web Components: emitowanie zdarzeń ([custom events](../05-dom-i-web-api/02-zdarzenia.md)).
- **Stan lokalny** — prywatne dane komponentu (otwarte/zamknięte, wartość inputa). Reguła: stan trzymaj **najniżej, jak się da**; podnoś (lifting state up) dopiero, gdy potrzebuje go rodzeństwo — wtedy wspólny przodek trzyma stan i rozdaje props + callbacki. Powyżej pewnej skali: [zarządzanie stanem](./04-zarzadzanie-stanem.md).

To jest **jednokierunkowy przepływ danych (one-way data flow)**: dane płyną w dół, sygnały w górę. Debugowanie: zawsze wiadomo, **kto jest właścicielem** danej informacji. (Two-way binding typu `v-model` to cukier: props + event w jednym zapisie.)

## Kompozycja zamiast dziedziczenia

UI buduje się przez **zagnieżdżanie i składanie**, nie przez `extends`:

- **Slots / children** — komponent-kontener przyjmuje „dziury" na treść (`<Card><UserInfo/></Card>`); layouty, modale, listy.
- **Komponenty wyspecjalizowane przez props** — `<Button variant="danger">` zamiast podklasy DangerButton.
- **Wydzielanie logiki bez UI** — hooki (React), composables (Vue), stores/runes (Svelte): reużywalna logika stanowa (np. `useFetch`, `useLocalStorage`) niezależna od renderowania. To współczesna odpowiedź na dawne mixiny/HOC.
- **Render props / scoped slots** — rodzic dostaje od dziecka dane do wyrenderowania własnej treści (odwrócenie kontroli).

Wzorce znane z [JS](../04-javascript/13-wzorce-projektowe-js.md) mają tu odpowiedniki: kompozycja funkcji, strategy przez props, observer przez subskrypcje stanu.

## Cykl życia i efekty uboczne

Komponent przechodzi fazy: **mount → update(y) → unmount**. Frameworki dają haki (lifecycle hooks / efekty), by wpiąć się w te momenty. Uniwersalne zasady:

1. **Render ma być czysty** — obliczenie widoku ze stanu, bez efektów ubocznych ([funkcje czyste](../04-javascript/02-funkcje.md)). Efekty (fetch, subskrypcje, timery, manipulacja DOM poza frameworkiem) — w dedykowanych hakach.
2. **Każda subskrypcja ma cleanup** — mount dodaje listener/timer/socket, unmount go zdejmuje. Brak symetrii = [memory leak](../04-javascript/12-zaawansowane.md) i duchy-handlery.
3. **Update'y bywają częste** — kosztowna praca w hakach aktualizacji wymaga warunków/memoizacji ([reaktywność](./03-modele-reaktywnosci.md)).

## Granice błędów i Suspense (koncepcyjnie)

- **Error boundary** — komponent-bezpiecznik: błąd renderu poddrzewa → fallback UI zamiast białego ekranu całej aplikacji ([strategia błędów](../04-javascript/11-obsluga-bledow.md)).
- **Suspense/async boundaries** — deklaratywne „czekam na dane/kod": fallback (skeleton) na czas ładowania; współgra z [code splittingiem](../09-wydajnosc/02-optymalizacja-ladowania.md) i [SSR streaming](../08-architektura-aplikacji/01-wzorce-renderowania.md).

Projektowanie granic (błędów, ładowania, danych) per widok/sekcja — to decyzja architektoniczna, nie techniczna ciekawostka.

## Projektowanie dobrych komponentów

- **Jedna odpowiedzialność**; komponent-strona komponuje mniejsze.
- **Kontrakt minimalny:** props tylko potrzebne; 15 propsów = sygnał do podziału lub kompozycji slotami.
- **Controlled vs uncontrolled** — stan formularza u rodzica (kontrolowany: pełna władza, więcej kodu) vs wewnątrz (niekontrolowany: prostota); świadomy wybór, nie przypadek ([formularze](../02-html/03-formularze.md)).
- **Prezentacyjne vs kontenerowe** (rozdzielenie danych od wyglądu) — dziś mniej dogmatyczne, ale rozdzielanie logiki (hook/composable) od szablonu wciąż porządkuje.
- Komponenty **design systemu** (Button, Modal) vs **domenowe** (KoszykPodsumowanie) — inne tempo zmian, inne miejsce w [strukturze projektu](../08-architektura-aplikacji/04-struktura-projektu.md).

## Pułapki i częste błędy

- Mutowanie props / sięganie do wnętrza dziecka — łamie kontrakt i przewidywalność.
- Stan globalny „bo wygodniej" tam, gdzie wystarczy lokalny + lifting.
- Efekty uboczne w renderze (fetch przy każdym przeliczeniu widoku).
- Brak cleanupów w unmount.
- Boskie komponenty 800-linijkowe — brak dekompozycji.
- Prop drilling przez 6 poziomów zamiast kompozycji (children/slots) albo kontekstu/stora — [zarządzanie stanem](./04-zarzadzanie-stanem.md).

## Pytania rekrutacyjne

1. **Wyjaśnij one-way data flow i jego zalety.** — props w dół, eventy w górę; przewidywalność, debugowalność.
2. **Jak komponenty komunikują się między sobą?** — rodzic-dziecko (props/eventy), rodzeństwo (lifting), dalekie (kontekst/store).
3. **Czym jest lifting state up i kiedy go stosujesz?** — wspólny przodek jako właściciel stanu.
4. **Controlled vs uncontrolled component?** — właściciel stanu formularza; kompromisy.
5. **Jak projektujesz granice komponentów?** — odpowiedzialność, kontrakt, reużywalność vs domena; error/loading boundaries.

## Dalsza lektura

- [React docs: Thinking in React](https://react.dev/learn/thinking-in-react) — uniwersalna metoda dekompozycji (czytaj koncepcyjnie)
- [Vue docs: Component Basics](https://vuejs.org/guide/essentials/component-basics) — ten sam model, inna składnia
- [Patterns.dev: Component patterns](https://www.patterns.dev/)
