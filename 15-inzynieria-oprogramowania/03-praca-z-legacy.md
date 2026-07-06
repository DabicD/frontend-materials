# Praca z legacy i refaktoryzacja

> **Poziom:** 🔴 zaawansowany (rzeczywistość seniora)
> **Wymagana wiedza:** [Czysty kod](./01-czysty-kod.md), [Testowanie](../13-testowanie/01-strategia-testowania.md), [Git](../14-narzedzia/01-git.md)

Prawda o pracy seniora: rzadko piszesz na zielonej trawie, częściej **pracujesz z istniejącym, niedoskonałym kodem**. Umiejętność bezpiecznego zmieniania systemu, którego nie rozumiesz w całości i nie możesz zatrzymać, odróżnia seniora od kogoś, kto potrafi tylko pisać nowe. To także najczęstszy temat rozmów senior+ („jak podszedłbyś do modernizacji...").

## Czym jest legacy i dług techniczny

- **Legacy** (wg M. Feathersa): kod **bez testów** — bo nie da się go bezpiecznie zmieniać. Nie chodzi o wiek ani technologię.
- **Dług techniczny** — metafora: skróty przyspieszają dziś, ale naliczają „odsetki" (wolniejszy rozwój) później. **Świadomy** dług (zapisany, z planem spłaty) jest narzędziem; **nieświadomy** (bałagan) to strata. Kategorie: przestarzałe zależności, brak testów, złe abstrakcje, duplikacja, martwy kod.
- Dług nie jest wrogiem — jak finansowy, bywa racjonalny (szybkie wejście na rynek); problem to dług **niespłacany i nieświadomy**.

## Zanim zmienisz: zrozum i zabezpiecz

1. **Zrozum** — czytaj kod, [git blame/log](../14-narzedzia/01-git.md) (dlaczego tak jest — często istniał powód), rozmawiaj z autorami, uruchamiaj i obserwuj. **Prawo Czestertona (płot):** nie usuwaj czegoś, dopóki nie zrozumiesz, czemu tam jest.
2. **Charakteryzuj testami (characterization tests):** zanim zmienisz kod bez testów, napisz testy opisujące **obecne** zachowanie (nawet jeśli błędne — dokumentujesz stan faktyczny). Dają siatkę bezpieczeństwa: refaktor nie zmieni zachowania, jeśli testy zostają zielone ([strategia](../13-testowanie/01-strategia-testowania.md)).
3. **Znajdź szwy (seams):** miejsca, gdzie da się wpiąć/podmienić zależność bez zmiany reszty — od nich zaczyna się testowalność (wstrzyknięcie zależności zamiast twardego importu).

## Refaktoryzacja — dyscyplina, nie chaos

**Refaktoryzacja = zmiana struktury BEZ zmiany zachowania.** Klucz: **małe kroki** z zielonymi testami po każdym; commituj często ([git](../14-narzedzia/01-git.md)).

- Nie mieszaj refaktoru ze zmianą funkcjonalną w jednym [PR](./02-code-review-i-wspolpraca.md) — recenzent nie odróżni „przeniosłem" od „zmieniłem logikę". Osobne commity/PR-y.
- Typowe operacje: wydzielenie funkcji/komponentu, nadanie nazw, usunięcie duplikacji, guard clauses, rozbicie god-modułu ([czysty kod](./01-czysty-kod.md)).
- Narzędzia: bezpieczne refaktory IDE, [TypeScript](../06-typescript/05-typescript-w-praktyce.md) (kompilator jako siatka), [testy](../13-testowanie/01-strategia-testowania.md).

## Strategie modernizacji dużej skali

**Strangler Fig** — najważniejszy wzorzec: nowy system **stopniowo oplata** stary, przejmując funkcje kawałek po kawałku, aż stary „uschnie". Zero „big-bang rewrite":

- Fasada/routing kieruje część ruchu do nowej implementacji, resztę do starej.
- Frontendowo: migracja per [trasa](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)/feature/komponent; stare i nowe współistnieją (np. wyspa nowego frameworka w starej stronie, [Module Federation](../08-architektura-aplikacji/03-microfrontends-i-monorepo.md)).
- **Dlaczego nie rewrite od zera:** utrata wiedzy zaklętej w starym kodzie (obsłużone edge case'y!), miesiące bez wartości biznesowej, ryzyko, że nowy powtórzy błędy starego. „Second-system effect" + zamrożenie rozwoju = klasyczna droga do porażki.

**Feature flags** — wdrażaj kod „uśpiony", włączaj stopniowo (canary, % użytkowników), natychmiast wyłącz przy problemie ([CI/CD](../16-devops-dla-frontendu/01-ci-cd.md)). Oddzielają **deploy** od **release**; kluczowe przy ryzykownych migracjach. Koszt: sprzątanie martwych flag.

**Boy Scout Rule** — „zostaw kod czystszym, niż zastałeś": drobne, oportunistyczne ulepszenia przy okazji pracy nad obszarem, zamiast wielkich akcji „posprzątamy wszystko" (które nigdy nie dostają priorytetu).

## Jak przekonać do spłaty długu (perspektywa senior)

- Mów **językiem biznesu:** nie „kod jest brzydki", lecz „ta część spowalnia każdą zmianę o X i generuje Y bugów/miesiąc".
- **Wiąż z funkcjami:** refaktor „przy okazji" dostarczania wartości łatwiej przejdzie niż osobny „sprint techniczny".
- **Mierz i priorytetyzuj:** obszary o największym ruchu zmian × największym bólu; nie refaktoruj kodu, którego nikt nie dotyka.
- **Małe, ciągłe** > wielkie i rzadkie (i tak odkładane).

## Pułapki i częste błędy

- **Big-bang rewrite** — kuszący, zwykle katastrofalny (utrata wiedzy, zamrożenie, przekroczenia).
- Refaktor **bez testów** charakteryzujących — „posprzątałem" = wprowadziłem regresje.
- Zmiana zachowania pod płaszczykiem refaktoru (i odwrotnie) w jednym PR.
- Usuwanie „dziwnego" kodu bez zrozumienia (płot Chestertona) — kasowanie obsłużonych edge case'ów.
- Wielkie refaktory naraz zamiast małych kroków — niereviewowalne, ryzykowne.
- Refaktoryzacja kodu, który zaraz zniknie/nikt nie rusza — praca bez zwrotu.
- Feature flags bez sprzątania — dług flag rośnie w nowy bałagan.

## Pytania rekrutacyjne

1. **Jak podejdziesz do zmiany w kodzie bez testów, którego nie rozumiesz?** — zrozum + characterization tests + małe kroki + szwy.
2. **Big-bang rewrite vs strangler fig — co i czemu?** — stopniowe przejmowanie; ryzyka rewrite (wiedza, czas, zamrożenie).
3. **Co to dług techniczny i jak nim zarządzasz?** — metafora, świadomy vs nie; mierzenie, priorytetyzacja, komunikacja biznesowa.
4. **Do czego feature flags w modernizacji?** — deploy≠release, canary, rollback; koszt sprzątania.
5. **Jak przekonasz PO/managera do refaktoru?** — język biznesu (koszt zmian/bugi), wiązanie z funkcjami, małe kroki.
6. **Czym jest reguła Chestertona i po co?** — nie usuwaj, zanim zrozumiesz cel; ochrona przed regresją.

## Dalsza lektura

- „Working Effectively with Legacy Code" (M. Feathers) — biblia tematu (szwy, characterization tests)
- [Martin Fowler: StranglerFigApplication](https://martinfowler.com/bliki/StranglerFigApplication.html) + „Refactoring" (książka)
- [martinfowler.com: TechnicalDebt](https://martinfowler.com/bliki/TechnicalDebt.html)
