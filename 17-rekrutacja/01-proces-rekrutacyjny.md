# Proces rekrutacyjny frontend developera

> **Poziom:** 🟢 wszystkie poziomy
> **Wymagana wiedza:** brak (meta-wiedza o rekrutacji)

Świetne umiejętności techniczne nie wystarczą, jeśli nie umiesz ich **pokazać** w procesie rekrutacyjnym. Ten plik to mapa typowego procesu i strategia przygotowania — od CV po negocjacje. Wiedzę merytoryczną dostarczają pozostałe sekcje; tu chodzi o to, jak ją zaprezentować.

## Etapy typowego procesu

1. **Screening CV** — automat/rekruter, sekundy na decyzję.
2. **Rozmowa z rekruterem (HR)** — dopasowanie do roli, oczekiwania, „miękkie" motywacje, widełki.
3. **Zadanie techniczne** — home assignment (projekt do domu) lub live coding ([zadania praktyczne](./03-zadania-praktyczne.md)).
4. **Rozmowa techniczna** — teoria + doświadczenie ([pytania teoretyczne](./02-pytania-teoretyczne.md)); dla senior często [system design](./04-frontend-system-design.md).
5. **Rozmowa behawioralna** — współpraca, konflikty, przywództwo (senior+); STAR (niżej).
6. **Rozmowa z zespołem/managerem** — kultura, dopasowanie, Twoje pytania.
7. **Oferta i negocjacje.**

Kolejność i liczba etapów różnią się (startup: 2–3 szybkie; korporacja: 5–7). Zapytaj rekrutera o przebieg — pokazuje profesjonalizm i pozwala się przygotować.

## CV i portfolio

- **CV zwięzłe (1–2 strony), skanowalne:** stack technologiczny, **osiągnięcia z liczbami** („skróciłem LCP z 4 s do 1,8 s", „zmigrowałem 200 komponentów na TS") zamiast listy obowiązków. Impact > technologie.
- **Dopasuj do oferty** — słowa kluczowe z ogłoszenia (screening bywa automatyczny), ale bez ściemy.
- **Portfolio/GitHub** — 2–3 dopracowane projekty > 20 porzuconych; działające demo + [README](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md) + czysty kod. Wkład w open source, blog, prezentacje — wyróżniają.
- **LinkedIn** aktualny; dla wielu ról to pierwsze źródło kandydatów.

## Jak przygotować się merytorycznie

- **Fundamenty ponad frameworkiem:** rozmowy sprawdzają [JS](../04-javascript/01-typy-i-zmienne.md) ([closures](../04-javascript/03-scope-hoisting-closures.md), [event loop](../04-javascript/09-event-loop.md), [this](../04-javascript/04-this-i-kontekst.md), async), [CSS](../03-css/01-kaskada-specyficznosc.md), [HTML/a11y](../10-dostepnosc/01-podstawy-a11y.md), sieć/[HTTP](../01-fundamenty-sieci/02-protokol-http.md) — bank pytań: [pytania teoretyczne](./02-pytania-teoretyczne.md).
- **Ćwicz na głos** — tłumaczenie konceptu komuś to inna umiejętność niż wiedza bierna; nagrywaj się lub ćwicz z kimś.
- **Przygotuj historie** z doświadczenia (trudny bug, decyzja architektoniczna, konflikt) — do pytań behawioralnych.
- **Poznaj firmę** — produkt, stack (oferta/StackShare), otwórz ich stronę w [DevTools](../14-narzedzia/05-devtools.md) (co ciekawego/do poprawy — świetny materiał na rozmowę).

## Pytania behawioralne — metoda STAR

Struktura odpowiedzi: **Situation → Task → Action → Result.**

> „Opowiedz o trudnym bugu." → kontekst (S), Twoje zadanie (T), **co konkretnie zrobiłeś Ty** (A — „ja", nie „my"), wynik z liczbą/wnioskiem (R).

Typowe pytania: konflikt w zespole, błędna decyzja i wniosek, dostarczanie pod presją, [feedback w code review](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md), przekonanie do zmiany technicznej. Senior: mentoring, wpływ na architekturę, [dług techniczny](../15-inzynieria-oprogramowania/03-praca-z-legacy.md), trade-offy.

## Poziomy: junior vs mid vs senior (czego się oczekuje)

- **Junior** — solidne fundamenty, chęć nauki, umiejętność pracy z zadaniem pod okiem; „jak myślisz" > „ile wiesz".
- **Mid** — samodzielność, znajomość ekosystemu, dobre praktyki, świadomość trade-offów.
- **Senior** — [architektura](../08-architektura-aplikacji/04-struktura-projektu.md), [system design](./04-frontend-system-design.md), decyzje i ich uzasadnienie, wpływ na zespół ([mentoring/review](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md)), komunikacja z biznesem, dowożenie. Głębia i kontekst, nie liczba znanych bibliotek.

## Twoje pytania do firmy (nie pomijaj!)

Rozmowa jest dwustronna — dobre pytania pokazują dojrzałość i chronią Cię przed złą decyzją: proces wdrożeń i [CI/CD](../16-devops-dla-frontendu/01-ci-cd.md), podejście do [testów](../13-testowanie/01-strategia-testowania.md)/jakości/[długu](../15-inzynieria-oprogramowania/03-praca-z-legacy.md), jak wygląda [code review](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md), on-call, rozwój/ścieżka awansu, jak wygląda typowy tydzień.

## Negocjacje (skrót)

- **Znaj widełki rynkowe** (raporty płacowe, społeczności) przed rozmową o pieniądzach.
- **Nie podawaj pierwszej liczby**, jeśli możesz; a jeśli musisz — kotwicz górą realnego zakresu.
- **Negocjuj całość** — nie tylko baza: bonus, sprzęt, budżet szkoleniowy, urlop, praca zdalna, ścieżka.
- **Miej ofertę/alternatywy** jako dźwignię; bądź gotów odejść od złej oferty. Ton: partnerski, nie roszczeniowy.

## Pułapki i częste błędy

- Uczenie się frameworka zamiast fundamentów — braki wychodzą w pierwszych pytaniach.
- Wiedza bierna bez ćwiczenia mówienia — „wiem, ale nie umiem wytłumaczyć".
- CV z obowiązkami zamiast osiągnięć; brak liczb.
- Zero pytań do firmy — sygnał braku zainteresowania.
- Odpowiedzi „my" zamiast „ja" w behawioralnych — nie widać Twojego wkładu.
- Blefowanie („znam X") — łatwe do zdemaskowania; lepiej „nie miałem okazji, ale rozumiem koncept i szybko się uczę".
- Zaniedbanie negocjacji („wezmę, co dają").

## Pytania rekrutacyjne (o samym procesie/sobie)

1. **Opowiedz o sobie** — 60–90 s: kim jesteś, kluczowe doświadczenie, czemu ta rola; przećwicz.
2. **Twoja największa porażka/błąd?** — szczery przykład + wniosek/zmiana (STAR).
3. **Czemu chcesz do nas / czemu odchodzisz?** — konkretnie o firmie/produkcie, bez narzekania na obecnego.
4. **Gdzie widzisz się za X lat / czego chcesz się uczyć?** — spójność z rolą.
5. **Masz pytania?** — zawsze tak; patrz wyżej.

## Dalsza lektura

- [Tech Interview Handbook](https://www.techinterviewhandbook.org/) — kompleksowo (w tym behawioralne, negocjacje)
- [GreatFrontEnd](https://www.greatfrontend.com/) — proces i przygotowanie pod frontend
- [levels.fyi](https://www.levels.fyi/) — dane płacowe do negocjacji
