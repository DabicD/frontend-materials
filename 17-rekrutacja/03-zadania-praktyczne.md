# Zadania praktyczne i live coding

> **Poziom:** 🟡 średni → 🔴
> **Wymagana wiedza:** [JavaScript](../04-javascript/02-funkcje.md), [DOM](../05-dom-i-web-api/01-dom-manipulacja.md), [asynchroniczność](../04-javascript/08-asynchronicznosc.md)

Część praktyczna rozmowy sprawdza to, czego teoria nie pokaże: jak **myślisz i kodujesz** pod obserwacją. Ocenia się proces (dopytywanie, dekompozycję, komunikację), nie tylko działający wynik. Ten plik to katalog typowych zadań frontendowych + strategia podejścia. Zadania „projektowe" (zaprojektuj autocomplete jako system) → [system design](./04-frontend-system-design.md).

## Formaty zadań

1. **Live coding (JS/utils)** — implementacja funkcji na współdzielonym edytorze/whiteboardzie. Frontend rzadko robi ciężkie algorytmy (LeetCode) — częściej **praktyczny JS**.
2. **Budowa komponentu/mini-appki** — na żywo lub jako home assignment (autocomplete, todo, licznik, tabela, formularz).
3. **Debugging** — dostajesz zepsuty kod, znajdź i napraw ([DevTools](../14-narzedzia/05-devtools.md)).
4. **Code review** — oceniasz fragment (senior; [code review](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md)).
5. **Home assignment** — projekt do domu; oceniany jak prawdziwy [PR](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md).

## Strategia live coding (proces > wynik)

1. **Dopytaj o wymagania ZANIM zaczniesz kodować** — wejścia/wyjścia, edge case'y, ograniczenia, oczekiwane API. Cisza i od razu kod to czerwona flaga; zadawanie pytań to zielona.
2. **Naszkicuj podejście na głos** — „zrobię to tak..., złożoność...". Myślenie na głos jest **oceniane** — rozmówca chce zobaczyć Twój tok rozumowania.
3. **Zacznij od działającego, prostego** rozwiązania (nawet naiwnego), potem ulepszaj — działające > eleganckie-niedokończone.
4. **Testuj po drodze** — przykładowe wejścia, edge case'y (pusta tablica, null, duplikaty).
5. **Komunikuj kompromisy** — „to O(n²), da się O(n) z Set kosztem pamięci".
6. **Zaciąłeś się?** — myśl głośno, poproś o podpowiedź; współpraca > udawanie. Utknięcie w ciszy jest gorsze niż pytanie.

## Klasyczne zadania JS (umieć z ręki)

Najczęstsze — powiązane z konkretną wiedzą w skarbnicy:

- **`debounce` i `throttle`** — absolutny hit → implementacja i różnice w [optymalizacji działania](../09-wydajnosc/03-optymalizacja-dzialania.md).
- **Event emitter** (`on`/`emit`/`off`) → [wzorce projektowe](../04-javascript/13-wzorce-projektowe-js.md).
- **`once`, `memoize`, `curry`, `pipe/compose`** — [closures](../04-javascript/03-scope-hoisting-closures.md), [funkcje](../04-javascript/02-funkcje.md).
- **Deep clone / deep equal** — rekursja, cykle, typy ([structuredClone](../04-javascript/07-skladnia-es6-plus.md)).
- **Własny `Promise.all` / `Promise` / retry / promise pool (limit współbieżności)** → [asynchroniczność](../04-javascript/08-asynchronicznosc.md).
- **`groupBy`, flatten, unikalizacja** → [kolekcje](../04-javascript/06-kolekcje-i-iteracja.md).
- **Polyfill `bind`/`map`/`reduce`** → [this](../04-javascript/04-this-i-kontekst.md), [prototypy](../04-javascript/05-obiekty-i-prototypy.md).

Wskazówka: te zadania testują **fundamenty** (closures, async, this, prototypy) w praktyce — dlatego cała [sekcja JS](../04-javascript/01-typy-i-zmienne.md) to Twój materiał treningowy.

## Klasyczne zadania komponentowe

- **Autocomplete/typeahead** — [debounce](../09-wydajnosc/03-optymalizacja-dzialania.md) + anulowanie ([AbortController](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)) + [race conditions](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md) + [a11y (combobox)](../10-dostepnosc/02-aria-i-semantyka.md) + klawiatura. Deceptively hard — patrz też [system design](./04-frontend-system-design.md).
- **Todo list** — CRUD, stan, filtrowanie ([zarządzanie stanem](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)).
- **Modal/dropdown** — [focus trap, Esc, a11y](../10-dostepnosc/03-a11y-w-praktyce.md), [click outside](../05-dom-i-web-api/02-zdarzenia.md).
- **Tabs/accordion** — [wzorzec ARIA + klawiatura](../10-dostepnosc/02-aria-i-semantyka.md).
- **Infinite scroll** — [IntersectionObserver](../05-dom-i-web-api/05-observery.md) + [paginacja](../12-komunikacja-z-api/01-rest.md).
- **Star rating / carousel / tic-tac-toe / stopwatch** — stan + zdarzenia + [dostępność](../10-dostepnosc/01-podstawy-a11y.md).

Ocena obejmuje: poprawność, **obsługę edge case'ów i stanów** (loading/error/empty — [wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)), **dostępność** (klawiatura!), czystość kodu.

## Home assignment — jak wyróżnić się pozytywnie

To Twoja próbka pracy — traktuj jak produkcyjny [PR](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md):

- **README** — jak uruchomić, decyzje i trade-offy, co byś dodał mając więcej czasu.
- **Zakres** — zrób dobrze wymagane, nie rozbudowuj bez sensu; jakość > ilość funkcji.
- **Podstawy jakości:** [typy](../06-typescript/01-podstawy-typowania.md), [sensowne testy](../13-testowanie/01-strategia-testowania.md) (choćby kluczowej logiki), [czysta struktura](../08-architektura-aplikacji/04-struktura-projektu.md), [czytelne commity](../14-narzedzia/01-git.md).
- **Nie pomijaj** stanów błędu/empty i [dostępności](../10-dostepnosc/03-a11y-w-praktyce.md) — to najczęściej różnicuje kandydatów.
- **Działające demo** + czytelny kod > przekombinowana architektura na małym zadaniu ([YAGNI](../15-inzynieria-oprogramowania/01-czysty-kod.md)).

## Pułapki i częste błędy

- Kodowanie od razu, bez dopytania o wymagania i edge case'y.
- Cisza podczas myślenia — rozmówca nie widzi Twojego procesu.
- Dążenie do „idealnego" rozwiązania i niedokończenie działającego.
- Pomijanie edge case'ów (pusta lista, null, błąd sieci) i stanów UI.
- Ignorowanie dostępności/klawiatury w zadaniach komponentowych.
- Home assignment „na odwal" albo przeciwnie — over-engineering prostego zadania.
- Brak testów/README w zadaniu domowym.
- Udawanie, że się wie, zamiast przyznania „nie jestem pewien, sprawdziłbym X".

## Pytania rekrutacyjne (o samym podejściu)

1. **Jak podchodzisz do nowego zadania?** — wymagania → szkic → działające MVP → iteracja → edge case'y/testy.
2. **Jak radzisz sobie, gdy nie znasz rozwiązania?** — dekompozycja, myślenie głośne, pytania, analogie.
3. **Co jest ważne w komponencie poza tym, że działa?** — stany (loading/error/empty), a11y, edge case'y, czytelność.

## Dalsza lektura

- [GreatFrontEnd — coding questions (utils + komponenty)](https://www.greatfrontend.com/questions)
- [BFE.dev](https://bigfrontend.dev/) — praktyczne zadania JS z rozmów
- [Frontend Interview Handbook — coding round](https://www.frontendinterviewhandbook.com/)
