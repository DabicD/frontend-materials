# Bank pytań teoretycznych

> **Poziom:** 🟢🟡🔴 wszystkie
> **Wymagana wiedza:** cała skarbnica — to plik-indeks

Pogrupowany zestaw pytań, które **realnie padają** na rozmowach frontendowych. Ten plik celowo **nie duplikuje odpowiedzi** — każde pytanie linkuje do pliku-domu, gdzie temat jest wyjaśniony (a na końcu każdego z tych plików jest własna sekcja „Pytania rekrutacyjne" z odpowiedziami). Używaj go jako **checklisty powtórkowej**: przeczytaj pytanie, odpowiedz na głos, sprawdź w linku.

Legenda: 🟢 podstawowy · 🟡 średni · 🔴 zaawansowany/senior.

## JavaScript — rdzeń (pytany zawsze)

- 🟢 Typy prymitywne vs referencyjne; wartość vs referencja; falsy; `==` vs `===` → [typy i zmienne](../04-javascript/01-typy-i-zmienne.md)
- 🟡 `var`/`let`/`const`, hoisting, TDZ → [typy i zmienne](../04-javascript/01-typy-i-zmienne.md), [scope](../04-javascript/03-scope-hoisting-closures.md)
- 🔴 **Czym jest closure?** (klasyk nad klasyki) → [scope, hoisting, closures](../04-javascript/03-scope-hoisting-closures.md)
- 🟡 Deklaracja vs wyrażenie vs arrow; czym różni się arrow → [funkcje](../04-javascript/02-funkcje.md)
- 🟡 **Od czego zależy `this`?** call/apply/bind → [this i kontekst](../04-javascript/04-this-i-kontekst.md)
- 🔴 Dziedziczenie prototypowe; co robi `new`; klasy jako cukier → [obiekty i prototypy](../04-javascript/05-obiekty-i-prototypy.md)
- 🔴 **Opisz event loop; co wypisze ten kod?** microtask vs macrotask → [event loop](../04-javascript/09-event-loop.md)
- 🟡 Promise (stany, łańcuch); async/await; `Promise.all/race/allSettled/any`; sekwencyjnie vs równolegle → [asynchroniczność](../04-javascript/08-asynchronicznosc.md)
- 🟡 `map/filter/reduce`; `Map`/`Set`/`WeakMap`; protokół iteracji → [kolekcje i iteracja](../04-javascript/06-kolekcje-i-iteracja.md)
- 🟡 `??` vs `||`; destrukturyzacja; spread vs rest; głęboka kopia → [składnia ES6+](../04-javascript/07-skladnia-es6-plus.md)
- 🟡 ESM vs CommonJS; dynamiczny import → [moduły](../04-javascript/10-moduly.md)
- 🔴 Garbage collection i memory leaki; Proxy; generatory → [zaawansowane](../04-javascript/12-zaawansowane.md)

## TypeScript

- 🟢 `any` vs `unknown`; co robi `strictNullChecks` → [podstawy typowania](../06-typescript/01-podstawy-typowania.md)
- 🟡 `interface` vs `type`; discriminated union; narrowing → [interfejsy i typy](../06-typescript/02-interfejsy-i-typy.md)
- 🟡 Po co generyki; `keyof`/`T[K]` → [funkcje i generyki](../06-typescript/03-funkcje-i-generyki.md)
- 🔴 `Pick`/`Partial`/`ReturnType` od zera; `infer` → [typy zaawansowane](../06-typescript/04-typy-zaawansowane.md)
- 🔴 Kto kompiluje a kto sprawdza typy; walidacja danych z API → [TS w praktyce](../06-typescript/05-typescript-w-praktyce.md)

## CSS i HTML

- 🟢 **Jak liczy się specificity?**; kaskada; `@layer` → [kaskada i specificity](../03-css/01-kaskada-specyficznosc.md)
- 🟢 `content-box` vs `border-box`; margin collapsing; `em` vs `rem` → [box model](../03-css/02-box-model-i-jednostki.md)
- 🟡 `position` (relative/absolute/fixed/sticky); **jak działa `z-index`/stacking context** → [układ dokumentu](../03-css/03-uklad-dokumentu.md)
- 🟡 **Flexbox vs Grid — kiedy co?**; `flex: 1`; `min-width: auto` → [flexbox](../03-css/04-flexbox.md), [grid](../03-css/05-grid.md)
- 🟡 Mobile-first; media vs container queries; `clamp()` → [responsywność](../03-css/06-responsywnosc.md)
- 🟡 Czemu `transform`/`opacity` są wydajne → [animacje](../03-css/07-animacje-i-transformacje.md), [jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)
- 🔴 Organizacja CSS w skali; BEM/Modules/Tailwind → [architektura CSS](../03-css/09-architektura-css.md)
- 🟢 Po co semantyka; `<a>` vs `<button>` → [semantyka](../02-html/02-semantyka.md)
- 🟡 `async` vs `defer`; `<!DOCTYPE>`; DOMContentLoaded vs load → [struktura dokumentu](../02-html/01-struktura-dokumentu.md)

## DOM, przeglądarka, sieć

- 🟡 `querySelectorAll` vs `getElementsByClassName`; atrybut vs property → [DOM](../05-dom-i-web-api/01-dom-manipulacja.md)
- 🟡 **Fazy propagacji zdarzeń**; event delegation; `preventDefault` vs `stopPropagation` → [zdarzenia](../05-dom-i-web-api/02-zdarzenia.md)
- 🟢 **Co się dzieje po wpisaniu URL?** → [jak działa internet](../01-fundamenty-sieci/01-jak-dziala-internet.md), [przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)
- 🟡 Metody HTTP i idempotencja; kody statusów; **jak działa cache HTTP/ETag** → [protokół HTTP](../01-fundamenty-sieci/02-protokol-http.md)
- 🟡 **Critical rendering path**; reflow vs repaint → [jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)
- 🟡 Kiedy fetch odrzuca Promise; jak anulować request → [fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)
- 🟡 Cookies vs localStorage vs IndexedDB → [przechowywanie danych](../05-dom-i-web-api/04-przechowywanie-danych.md)
- 🟡 IntersectionObserver — lazy loading/infinite scroll → [obserwery](../05-dom-i-web-api/05-observery.md)
- 🔴 Async vs Web Worker → [web workers](../05-dom-i-web-api/06-web-workers.md)

## Frameworki i architektura

- 🟢 Jaki problem rozwiązują frameworki; deklaratywne vs imperatywne → [po co frameworki](../07-frameworki-koncepcyjnie/01-po-co-frameworki.md)
- 🟡 One-way data flow; lifting state; controlled vs uncontrolled → [model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)
- 🔴 **Jak działa Virtual DOM?**; signals vs VDOM; czemu `key` nie index → [modele reaktywności](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md)
- 🔴 **Gdzie żyje stan?**; server state vs client state; Flux/Redux → [zarządzanie stanem](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md)
- 🟡 Jak działa routing SPA; czemu 404 po odświeżeniu → [routing kliencki](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)
- 🔴 **CSR/SSR/SSG/ISR — kompromisy**; hydracja → [wzorce renderowania](../08-architektura-aplikacji/01-wzorce-renderowania.md)
- 🟡 SPA vs MPA; cykl życia Service Workera → [SPA/MPA/PWA](../08-architektura-aplikacji/02-spa-mpa-pwa.md)
- 🔴 Jak organizujesz duży projekt; granice modułów → [struktura projektu](../08-architektura-aplikacji/04-struktura-projektu.md)

## Wydajność

- 🟡 **Core Web Vitals** (LCP/INP/CLS); lab vs field → [core web vitals](../09-wydajnosc/01-core-web-vitals.md)
- 🟡 Strona ładuje się wolno — plan; preload vs prefetch; code splitting → [optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md)
- 🔴 **Napisz debounce i throttle**; layout thrashing; wirtualizacja listy → [optymalizacja działania](../09-wydajnosc/03-optymalizacja-dzialania.md)
- 🔴 Warstwy cache; stale-while-revalidate → [strategie cache'owania](../09-wydajnosc/04-strategie-cachowania.md)

## Dostępność i bezpieczeństwo

- 🟢 Czym jest WCAG/poziom AA; jak niewidomi korzystają ze stron → [podstawy a11y](../10-dostepnosc/01-podstawy-a11y.md)
- 🟡 Pierwsza zasada ARIA; accessibility tree; live regions → [ARIA i semantyka](../10-dostepnosc/02-aria-i-semantyka.md)
- 🟡 Dostępny modal (focus trap); wymogi kontrastu → [a11y w praktyce](../10-dostepnosc/03-a11y-w-praktyce.md)
- 🔴 **Czym jest XSS i jak się bronisz?**; CSRF; supply chain → [ataki na frontend](../11-bezpieczenstwo/01-ataki-na-frontend.md)
- 🔴 **SOP i CORS — kto konfiguruje?**; preflight; CSP → [mechanizmy obronne](../11-bezpieczenstwo/02-mechanizmy-obronne.md)
- 🔴 **Gdzie trzymać token** (XSS vs CSRF); OAuth/PKCE; czemu autoryzacja we froncie to tylko UX → [uwierzytelnianie](../11-bezpieczenstwo/03-uwierzytelnianie-i-autoryzacja.md)

## API, testy, narzędzia, inżynieria

- 🟡 Co czyni API RESTowym; paginacja offset vs cursor → [REST](../12-komunikacja-z-api/01-rest.md)
- 🟡 Jaki problem REST rozwiązuje GraphQL; czemu cache trudniejszy → [GraphQL](../12-komunikacja-z-api/02-graphql.md)
- 🟡 Polling vs SSE vs WebSocket → [czas rzeczywisty](../12-komunikacja-z-api/03-czas-rzeczywisty.md)
- 🔴 **Race condition w data fetchingu**; server state; optimistic updates → [wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)
- 🟡 Piramida vs trofeum; czego nie testować → [strategia testowania](../13-testowanie/01-strategia-testowania.md)
- 🟡 Mock/stub/spy; TDD; `toBe` vs `toEqual` → [testy jednostkowe](../13-testowanie/02-testy-jednostkowe.md)
- 🟡 Zasada Testing Library; czemu selektory po roli → [testy komponentów](../13-testowanie/03-testy-komponentow-i-integracyjne.md)
- 🟡 **Merge vs rebase**; revert vs reset; reflog → [git](../14-narzedzia/01-git.md)
- 🟡 Lockfile i `npm ci`; `^` vs `~`; peerDependencies → [pakiety](../14-narzedzia/02-pakiety-i-zaleznosci.md)
- 🟡 Co robi bundler; tree shaking; czemu Vite szybki → [bundlery](../14-narzedzia/03-bundlery-i-proces-budowania.md)
- 🔴 Czysty kod; DRY nie zawsze; SOLID we froncie → [czysty kod](../15-inzynieria-oprogramowania/01-czysty-kod.md)
- 🔴 Praca z legacy; strangler fig vs rewrite; dług techniczny → [praca z legacy](../15-inzynieria-oprogramowania/03-praca-z-legacy.md)
- 🟡 CI/CD; preview deployments; feature flags → [CI/CD](../16-devops-dla-frontendu/01-ci-cd.md)
- 🟡 Skąd wiesz, że produkcja działa; RUM → [monitoring](../16-devops-dla-frontendu/03-monitoring-i-obserwabilnosc.md)

## Jak używać tej listy (strategia powtórki)

1. **Przejdź „na głos"** — dla każdego pytania spróbuj odpowiedzieć bez zaglądania; dopiero potem link.
2. **Oznacz luki** — pytania, przy których się zaciąłeś → przeczytaj cały plik-dom, nie tylko sekcję pytań.
3. **Priorytet:** rzeczy oznaczone „klasyk"/pogrubione i 🔴 padają najczęściej na senior; 🟢 to must-know absolutne.
4. **Ćwicz zadania praktyczne osobno** → [zadania praktyczne](./03-zadania-praktyczne.md) i [system design](./04-frontend-system-design.md).
5. **Nie kuj odpowiedzi** — rozmówca drąży „a dlaczego?"; rozumienie z plików-domów obroni się przy dopytywaniu, wykuta formułka nie.

## Dalsza lektura

- Sekcje „Pytania rekrutacyjne" na końcu **każdego** pliku w tej skarbnicy (z odpowiedziami)
- [GreatFrontEnd — Front End Interview Playbook](https://www.greatfrontend.com/front-end-interview-playbook)
- [JavaScript Questions (lydiahallie)](https://github.com/lydiahallie/javascript-questions)
