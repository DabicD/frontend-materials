# Code review i współpraca

> **Poziom:** 🔴 zaawansowany (kluczowe dla senior/lead)
> **Wymagana wiedza:** [Git](../14-narzedzia/01-git.md), [Czysty kod](./01-czysty-kod.md)

Umiejętności techniczne wystarczą na juniora; senior jest oceniany też przez pryzmat **jak podnosi zespół**. Code review, jakość komunikacji w PR, dokumentacja decyzji — to praca, która skaluje się na innych. Częsty obszar pytań behawioralnych na rozmowach senior+.

## Po co code review

1. **Jakość i wychwytywanie błędów** — druga para oczu (choć automaty — [testy](../13-testowanie/01-strategia-testowania.md), [lint](../14-narzedzia/04-jakosc-kodu.md), TS — powinny łapać rzeczy mechaniczne, by review skupiło się na *projektowaniu*).
2. **Dzielenie się wiedzą** — o zmianach dowiaduje się więcej niż jedna osoba (bus factor); młodsi uczą się od starszych i odwrotnie.
3. **Spójność** — pilnowanie konwencji i [granic architektury](../08-architektura-aplikacji/04-struktura-projektu.md).
4. **Współodpowiedzialność** — kod należy do zespołu, nie autora.

## Jak być dobrym recenzentem

**Na czym się skupić (hierarchia):**
1. **Poprawność** — czy działa, edge case'y, błędy, [race conditions](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md), [bezpieczeństwo](../11-bezpieczenstwo/01-ataki-na-frontend.md).
2. **Projekt/architektura** — czy pasuje do systemu, właściwe abstrakcje, [granice](../08-architektura-aplikacji/04-struktura-projektu.md), testowalność.
3. **Czytelność/utrzymanie** — nazwy, złożoność ([czysty kod](./01-czysty-kod.md)).
4. **Testy** — czy pokrywają zachowanie, nie implementację.
5. Styl/formatowanie — **zostaw automatom**; ręczne czepianie się przecinków to marnowanie review.

**Jak komunikować:**
- **Krytykuj kod, nie osobę:** „ta funkcja robi dwie rzeczy" > „źle to napisałeś".
- **Pytaj zamiast rozkazywać:** „co sądzisz o wydzieleniu tego?" > „wydziel to" — zaprasza do dyskusji, uczy.
- **Oznaczaj wagę:** rozróżniaj blokery od sugestii (konwencja `nit:` dla drobiazgów, `question:`, `blocking:`); nie blokuj PR-a preferencją.
- **Doceniaj** dobre rozwiązania — review to nie tylko wytykanie.
- **Wyjaśniaj *dlaczego*** — komentarz uczący („to spowoduje re-render, bo...") > sam nakaz.
- **Bądź szybki** — zalegające PR-y blokują ludzi; szybki, częściowy feedback > idealny za trzy dni.

## Jak być dobrym autorem PR

- **Małe PR-y** — 200–400 linii recenzuje się rzetelnie; 2000-liniowy dostaje „LGTM" bez czytania. Dziel pracę.
- **Dobry opis:** *co* i **dlaczego** (kontekst, decyzje, trade-offy), jak testować, screenshoty/nagrania dla zmian UI, link do zadania.
- **Self-review przed wysłaniem** — przeczytaj własny diff; złap oczywiste, dodaj komentarze wyjaśniające trudne miejsca.
- **Czysty diff** — bez przypadkowych zmian formatowania/plików ([git add -p](../14-narzedzia/01-git.md)); refaktor osobno od zmiany funkcjonalnej.
- **Reaguj bez defensywności** — feedback dotyczy kodu; dyskutuj rzeczowo, nie broń ego. Nie zgadzasz się — argumentuj, ewentualnie eskaluj rozmową, nie wojną w komentarzach.
- **Zielony [CI](../16-devops-dla-frontendu/01-ci-cd.md)** przed prośbą o review — nie marnuj czasu recenzenta na to, co złapie automat.

## Współpraca i dokumentacja

- **ADR (Architecture Decision Record)** — krótki dokument: kontekst, decyzja, rozważane opcje, konsekwencje. Zapisuje **dlaczego** (za rok nikt nie pamięta) — bezcenne przy [pracy z legacy](./03-praca-z-legacy.md). Format: 1 strona, w repo.
- **README/CONTRIBUTING** — jak uruchomić, konwencje, gdzie co jest; onboarding nowych.
- **Dokumentacja komponentów** — Storybook (żywe przykłady, warianty, stany — także [testy wizualne](../13-testowanie/04-testy-e2e-i-specjalistyczne.md)).
- **Definition of Done** — wspólne kryteria (testy, a11y, review, dokumentacja) — kończy dyskusje „czy gotowe".
- **Pair/mob programming** — review „na żywo" dla trudnych problemów; szybszy transfer wiedzy niż async.
- **Wybór technologii/konwencji** — decyzja zespołowa (RFC/ADR), nie narzucona po cichu.

## Pułapki i częste błędy

- Nitpicking stylu, który powinien robić [Prettier/ESLint](../14-narzedzia/04-jakosc-kodu.md); pomijanie architektury.
- Wielkie PR-y → powierzchowne review → bugi przechodzą.
- Rubber-stamp „LGTM" bez czytania — pozorne review, realne ryzyko.
- Ton protekcjonalny/personalny → defensywność, spadek morale.
- Blokowanie PR-a osobistą preferencją zamiast argumentu.
- Brak zapisu decyzji → te same dyskusje w kółko, „archeologia" przez [git blame](../14-narzedzia/01-git.md).
- Autor ignorujący feedback lub broniący się emocjonalnie.

## Pytania rekrutacyjne

1. **Na czym skupiasz się w code review?** — poprawność→projekt→czytelność→testy; styl do automatów.
2. **Jak dajesz trudny feedback?** — o kodzie, pytania nie rozkazy, waga komentarzy, wyjaśnianie dlaczego.
3. **Jak przygotowujesz PR do review?** — mały, opis z „dlaczego", self-review, zielony CI, screenshoty.
4. **Autor nie zgadza się z Twoją uwagą — co robisz?** — argumenty, dyskusja, ewentualnie rozmowa/eskalacja; nie ego.
5. **Co to ADR i po co?** — zapis decyzji z kontekstem; unikanie utraty „dlaczego".
6. **Jak podnosisz jakość pracy zespołu?** — review, mentoring, konwencje, DoD, dokumentacja, automatyzacja.

## Dalsza lektura

- [Google Engineering Practices: Code Review](https://google.github.io/eng-practices/review/)
- [Conventional Comments](https://conventionalcomments.org/) — oznaczanie wagi uwag
- [ADR (Architecture Decision Records)](https://adr.github.io/)
