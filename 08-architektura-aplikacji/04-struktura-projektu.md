# Struktura projektu i granice modułów

> **Poziom:** 🔴 zaawansowany (codzienna praca seniora)
> **Wymagana wiedza:** [Moduły](../04-javascript/10-moduly.md), [Model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)

Struktura katalogów to nie kosmetyka — to **architektura zależności**: co może importować co. Zła struktura objawia się tak: każda zmiana dotyka 12 plików w 6 katalogach, nikt nie wie, gdzie położyć nowy kod, a „wspólne" moduły importują wszystko nawzajem. Dobra struktura czyni **zależności jawnymi i jednokierunkowymi**.

## Dwa podejścia

**Warstwowe (по typie plików):**

```
src/
├── components/   # wszystkie komponenty
├── hooks/        # wszystkie hooki
├── services/     # całe API
├── utils/
└── types/
```

Działa w małych projektach; w dużych **featura rozsmarowana po 5 katalogach** — zmiana „koszyka" dotyka components/, hooks/, services/… a katalogi rosną w setki plików bez właścicieli.

**Feature-based (po domenie):**

```
src/
├── app/                 # bootstrap: routing, providery, konfiguracja
├── features/            # moduły domenowe
│   ├── cart/
│   │   ├── components/  # prywatne komponenty koszyka
│   │   ├── api.ts       # endpointy koszyka
│   │   ├── store.ts     # stan koszyka
│   │   └── index.ts     # PUBLICZNE API modułu
│   ├── catalog/
│   └── auth/
└── shared/              # naprawdę wspólne: ui-kit, utils, api-client
```

Kod, który zmienia się razem, mieszka razem (kolokacja); moduł ma granicę i właściciela. Formalizacja tego podejścia z dodatkowymi warstwami: **Feature-Sliced Design** (app→pages→widgets→features→entities→shared).

## Granice modułów — sedno tematu

Katalog staje się **modułem**, gdy ma:

1. **Publiczne API** — `index.ts` re-eksportuje to, co wolno używać z zewnątrz; reszta jest prywatna:

```ts
// features/cart/index.ts — JEDYNE wejście do modułu
export { CartWidget } from './components/CartWidget';
export { useCart } from './store';
// wnętrza (helpery, podkomponenty) — niedostępne
```

2. **Reguły kierunku zależności** (egzekwowane [ESLint-em](../14-narzedzia/04-jakosc-kodu.md): `import/no-restricted-paths`, boundaries, dependency-cruiser):
   - `features/*` mogą importować `shared`, **nigdy siebie nawzajem** (a przynajmniej: jawnie i rzadko),
   - `shared` nie importuje z `features` (kierunek w dół zakazany),
   - `app` składa wszystko (composition root).

Bez egzekwowania reguły umierają w tydzień. Cykl importów = sygnał złych granic ([moduły](../04-javascript/10-moduly.md)).

**Komunikacja między featurami:** przez `app` (kompozycja: strona składa CartWidget i CatalogList), przez `shared` (wspólne typy/eventy), przez [stan/URL](../07-frameworki-koncepcyjnie/04-zarzadzanie-stanem.md) — nie przez sięganie w bebechy sąsiada.

## Warstwa danych i separacja od UI

Uniwersalna zasada niezależna od struktury: **UI nie zna szczegółów transportu**. Komponent woła `useProducts()` / `productsApi.list()`, a nie klei URL-i i nie parsuje odpowiedzi:

```
komponenty → hooki/fasady domenowe → api-client (fetch, auth, błędy) → sieć
```

- Wymiana REST→GraphQL, zmiana kształtu odpowiedzi (mapowanie DTO→model domenowy), mockowanie w testach — dotykają jednej warstwy ([adapter/fasada](../04-javascript/13-wzorce-projektowe-js.md), [warstwa API](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).
- Typy domenowe ≠ typy DTO z API — mapuj na granicy, waliduj ([TS w praktyce](../06-typescript/05-typescript-w-praktyce.md)).

## Heurystyki praktyczne

- **Zaczynaj płasko, wydzielaj pod presją** — struktura na wyrost dla 10 plików to LARP; refaktor do features, gdy widzisz duplikację i rozproszenie.
- **Kolokacja testów i stylów** z kodem (`Button.tsx`, `Button.test.tsx`, `Button.module.css`).
- **shared/ tylko za dowodem** — drugi-trzeci użytkownik kodu, nie „może się przyda"; inaczej shared staje się śmietnikiem sprzęgającym wszystko.
- **Nazwy domenowe, nie techniczne** — `checkout/`, nie `forms2/`; struktura ma opowiadać, czym jest produkt (screaming architecture).
- Konsekwencja > doskonałość: jedna konwencja w zespole, spisana ([code review](../15-inzynieria-oprogramowania/02-code-review-i-wspolpraca.md)).

## Pułapki i częste błędy

- Importy w poprzek granic „bo szybciej" — po roku wszystko zależy od wszystkiego.
- Barrel files re-eksportujące pół projektu — [tree shaking i cykle](../04-javascript/10-moduly.md).
- utils.ts / helpers.ts jako rosnący worek bez granic.
- Głębokie zagnieżdżenia katalogów (5+ poziomów) — nawigowalność spada, importy `../../../../`.
- Struktura odzwierciedlająca framework zamiast domeny.
- Brak decyzji „gdzie kładziemy X" w zespole — każdy PR to nowa konwencja.

## Pytania rekrutacyjne

1. **Jak organizujesz duży projekt frontendowy?** — feature-based + shared + app; publiczne API modułów; egzekwowanie granic.
2. **Czym jest publiczne API modułu i po co?** — kontrolowana powierzchnia, swoboda refaktoru wnętrza.
3. **Jak zapobiegasz cyklom i imporcie „w poprzek"?** — reguły kierunku + lint/dependency-cruiser.
4. **Gdzie mieszka logika API i czemu nie w komponentach?** — warstwa danych; wymienialność, testowalność, spójna obsługa błędów.
5. **Kiedy wydzielasz kod do shared?** — po drugim użyciu, z właścicielem i kontraktem.

## Dalsza lektura

- [Feature-Sliced Design](https://feature-sliced.design/)
- [Bulletproof React — struktura projektu](https://github.com/alan2207/bulletproof-react) (koncepty przenośne poza React)
- [dependency-cruiser](https://github.com/sverweij/dependency-cruiser)
