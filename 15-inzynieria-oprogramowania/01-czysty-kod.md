# Czysty kod i zasady projektowe

> **Poziom:** 🔴 zaawansowany (poziom senior)
> **Wymagana wiedza:** [Funkcje](../04-javascript/02-funkcje.md), [Wzorce projektowe](../04-javascript/13-wzorce-projektowe-js.md), [Struktura projektu](../08-architektura-aplikacji/04-struktura-projektu.md)

Kod czyta się **wielokrotnie częściej niż pisze** — czytelność to nie estetyka, lecz ekonomia. „Czysty kod" to nie zbiór sztywnych reguł, ale zestaw heurystyk służących jednemu celowi: **łatwości zmiany**. Senior odróżnia zasadę od dogmatu i wie, kiedy pragmatyzm bije czystość.

## Nazewnictwo — najtańsza dźwignia czytelności

- **Nazwy ujawniają intencję:** `daysUntilExpiry`, nie `d`; `activeUsers`, nie `list`. Czytający rozumie bez komentarza.
- **Bez szumu i skrótów:** `user`, nie `userObj`/`usr`; `remove`, nie `rm`.
- **Konwencje:** funkcje = czasowniki (`calculateTotal`, `fetchUser`), boolean = `is/has/should` (`isLoading`, `hasPermission`), stałe UPPER_CASE, komponenty PascalCase.
- **Spójność w projekcie** — jedno pojęcie, jedna nazwa (`fetch`/`get`/`load` — wybierz jedno dla tej samej operacji).
- Dobra nazwa **eliminuje komentarz**; potrzeba komentarza „co to robi" to często sygnał złej nazwy.

## Funkcje

- **Jedna odpowiedzialność** — funkcja robi jedną rzecz i ma jeden poziom abstrakcji (nie miesza logiki biznesowej z manipulacją DOM i fetchowaniem).
- **Krótkie** — długa funkcja zwykle ukrywa kilka mniejszych; dziel wg sensu, nie sztywnego limitu linii.
- **Mało argumentów** — 0–2 idealnie; 4+ → obiekt parametrów (i tak czytelniejsze niż kolejność `fn(true, false, null)`); unikaj **boolean flag** (sygnał, że funkcja robi dwie rzeczy).
- **Czystość gdzie się da** ([funkcje czyste](../04-javascript/02-funkcje.md)) — efekty na brzegach; testowalność.
- **Bez side effects ukrytych w nazwie** — `getUser()`, które po cichu zapisuje coś do cache, kłamie.
- **Wczesne wyjścia (guard clauses)** zamiast piramid `if/else` — płaski kod czyta się liniowo.

## Zasady, które warto znać (i nie fetyszyzować)

- **DRY** (Don't Repeat Yourself) — jedno źródło prawdy dla wiedzy. Uwaga: **przedwczesna abstrakcja gorsza niż duplikacja** — dwie podobne rzeczy dziś mogą jutro rozejść się; nie łącz kodu, który jest *przypadkowo* podobny (reguła „rule of three": abstrahuj po trzecim wystąpieniu).
- **KISS** / **YAGNI** — najprostsze działające rozwiązanie; nie buduj na zapas („będzie potrzebne") — zwykle nie będzie, a złożoność zostaje.
- **Separation of Concerns** — [warstwa danych oddzielona od UI](../08-architektura-aplikacji/04-struktura-projektu.md), logika od prezentacji.
- **Composition over inheritance** — [kompozycja](../04-javascript/05-obiekty-i-prototypy.md) zamiast głębokich hierarchii.
- **Least astonishment** — kod robi to, czego czytający się spodziewa; brak niespodzianek.

## SOLID w kontekście frontendu

SOLID powstał dla OOP, ale duch przenosi się na komponenty/moduły:

- **S — Single Responsibility:** komponent/moduł ma jeden powód do zmiany. „God component" 800 linii łączący fetch, logikę i UI łamie to ([model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)).
- **O — Open/Closed:** rozszerzalne bez modyfikacji — np. warianty przez props/[strategię (mapa funkcji)](../04-javascript/13-wzorce-projektowe-js.md) zamiast rozrastającego się `if/else`.
- **L — Liskov:** komponent-wariant powinien być użyteczny wszędzie tam, gdzie bazowy (spójny kontrakt props).
- **I — Interface Segregation:** wąskie props/API — komponent nie wymaga 20 propsów, z których używa 3.
- **D — Dependency Inversion:** zależ od abstrakcji — UI woła `useProducts()`/interfejs repo, nie konkretny `fetch` ([warstwa API](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)); ułatwia testy i wymianę.

Na rozmowie: umieć podać **frontendowy przykład** każdej, nie recytować definicję.

## Code smells — sygnały ostrzegawcze

- Długie funkcje/komponenty, głębokie zagnieżdżenia (`if` w `if` w `if`).
- Duplikacja logiki (nie stylu).
- **Magic numbers/strings** (`if (status === 3)`) → nazwane stałe/enumy ([unie literałów](../06-typescript/02-interfejsy-i-typy.md)).
- Boolean flags jako argumenty; długie listy parametrów.
- Prop drilling przez wiele poziomów ([zarządzanie stanem](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)).
- Komentarze tłumaczące *co* robi kod (zamiast *dlaczego*) — zwykle znak, że kod jest niejasny.
- Zakomentowany kod (od tego jest [git](../14-narzedzia/01-git.md)).

## Komentarze i dokumentacja

- Dobry komentarz wyjaśnia **dlaczego** (decyzja, obejście, kontekst biznesowy), nie **co** (to mówi kod).
- Samodokumentujący kod > komentarze, które się dezaktualizują.
- Publiczne API/nietrywialne funkcje: JSDoc/typy ([TS](../06-typescript/01-podstawy-typowania.md) jako żywa dokumentacja).
- Decyzje architektoniczne: [ADR](./02-code-review-i-wspolpraca.md).

## Pragmatyzm — cecha seniora

Czysty kod służy **dostarczaniu wartości**, nie odwrotnie. Kontekst decyduje: prototyp/spike ≠ kod krytycznej ścieżki; „wystarczająco dobre i dowiezione" bije „idealne i spóźnione". Świadomy [dług techniczny](./03-praca-z-legacy.md) (zapisany, z planem) jest OK; nieświadomy — nie. Rozpoznawaj, kiedy dalsze polerowanie to już strata (over-engineering).

## Pułapki i częste błędy

- Dogmatyczne DRY → przedwczesne, błędne abstrakcje trudniejsze do rozplątania niż duplikacja.
- Over-engineering pod wymyślone przyszłe wymagania (łamanie YAGNI).
- „Sprytny" kod (jednoliniowce, popisy) zamiast czytelnego — koszt ponoszą kolejni czytelnicy.
- SOLID/wzorce aplikowane mechanicznie, mnożące pliki bez wartości.
- Mylenie „czysty" z „idealny" — paraliż zamiast dostarczania.

## Pytania rekrutacyjne

1. **Co dla Ciebie znaczy czysty kod?** — czytelność→łatwość zmiany; nazwy, małe funkcje, SoC; pragmatyzm.
2. **DRY — zawsze dobre?** — nie: przedwczesna abstrakcja, rule of three, przypadkowa vs prawdziwa duplikacja.
3. **Podaj frontendowy przykład zasady SOLID.** — np. Dependency Inversion: hook/interfejs zamiast fetch w komponencie.
4. **Jakie code smells rozpoznajesz najpierw?** — god component, duplikacja, magic values, prop drilling, boolean flags.
5. **Kiedy świadomie odpuszczasz czystość?** — prototyp/deadline; świadomy, zapisany dług vs bałagan.

## Dalsza lektura

- „Clean Code" (R. C. Martin) — z krytycznym okiem; oraz „A Philosophy of Software Design" (J. Ousterhout, bardziej wyważona)
- [Refactoring.guru: Code smells](https://refactoring.guru/refactoring/smells)
- [Kent C. Dodds: AHA Programming](https://kentcdodds.com/blog/aha-programming) (o przedwczesnej abstrakcji)
