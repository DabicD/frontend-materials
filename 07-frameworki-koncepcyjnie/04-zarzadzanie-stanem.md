# Zarządzanie stanem

> **Poziom:** 🔴 zaawansowany (decyzje architektoniczne)
> **Wymagana wiedza:** [Model komponentowy](./02-model-komponentowy.md), [Modele reaktywności](./03-modele-reaktywnosci.md)

„State management" to nie „którą bibliotekę wybrać", tylko **taksonomia stanu**: różne rodzaje stanu mają różne cykle życia i różne właściwe narzędzia. Największy błąd to jeden wielki store na wszystko — i właśnie od rozróżnienia rodzajów zaczyna się odpowiedź seniora.

## Taksonomia stanu (najważniejsza rzecz w tym pliku)

| Rodzaj | Przykłady | Właściwe miejsce |
|---|---|---|
| **Stan lokalny UI** | otwarty dropdown, wartość inputa, tab | komponent (useState/ref/signal) |
| **Stan współdzielony UI (client state)** | motyw, otwarty koszyk-panel, multi-step form | lifting → kontekst/store |
| **Stan serwera (server state)** | dane z API: produkty, profil, lista zamówień | **cache zapytań** (React Query/SWR-podobne), NIE ręczny store |
| **Stan w URL** | strona paginacji, filtry, otwarty modal szczegółów | **URL/query params** ([routing](./05-routing-po-stronie-klienta.md)) |
| **Stan formularzy** | wartości, błędy, dirty | biblioteka formularzy / lokalnie |
| **Stan trwały** | preferencje, drafty | [localStorage/IndexedDB](../05-dom-i-web-api/04-przechowywanie-danych.md) + synchronizacja |

**Server state ≠ client state:** dane z API są **cudzą własnością** — mają problemy cache'owania: świeżość, rewalidacja, deduplikacja requestów, retry, optimistic updates ([wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md)). Wrzucenie ich do Reduxa/store'a oznacza ręczne przepisanie tych mechanizmów. Biblioteki query-cache robią to za Ciebie — to one „zabiły" połowę dawnych zastosowań Reduxa.

**Stan w URL** — niedoceniany: filtry/paginacja w query params dają za darmo: link do wysłania, back/forward, odświeżenie bez utraty, SSR.

## Wzorzec store (Flux/Redux i potomkowie)

Dla **globalnego client state** wzorzec jednokierunkowy:

```
widok ──dispatch(action)──▶ store (reducer: (state, action) → newState) ──subscribe──▶ widok
```

- **Single source of truth**, zmiany tylko przez akcje, reducery czyste ([funkcje czyste](../04-javascript/02-funkcje.md)) — stąd: podróż w czasie (devtools), przewidywalność, testowalność ([observer](../04-javascript/13-wzorce-projektowe-js.md) pod spodem).
- Współczesne wcielenia są lżejsze: **Zustand/Pinia** (store bez boilerplate'u), **Jotai/signals** (atomy — stan złożony z małych niezależnych kawałków), Redux Toolkit (gdy Redux, to tylko RTK).
- **Selektory** — komponent subskrybuje wycinek stanu (`useStore(s => s.cart.count)`), nie całość: mniej renderów ([reaktywność](./03-modele-reaktywnosci.md)).
- Stan **normalizuj** jak bazę danych (encje po id + listy id), zamiast zagnieżdżonych drzew — aktualizacje punktowe, brak duplikatów:

```js
{ products: { byId: { 42: {...} }, allIds: [42, 7] }, cart: { itemIds: [42] } }
```

## Kontekst / dependency injection widoku

Mechanizmy typu React Context / Vue provide-inject rozwiązują **prop drilling** (przekazywanie props przez wiele poziomów). To kanał dystrybucji, **nie manager stanu**: zmiana wartości kontekstu odświeża wszystkich konsumentów. Dobre dla rzadko zmiennych: motyw, locale, bieżący użytkownik, zależności (klient API). Częste zmiany → store z selektorami.

## Zasady projektowe (uniwersalne)

1. **Minimalizuj stan:** wszystko, co policzalne z innego stanu — licz (computed/memo), nie przechowuj ([reaktywność](./03-modele-reaktywnosci.md)).
2. **Kolokacja:** stan możliwie blisko użycia; globalny to ostateczność, nie default.
3. **Pojedynczy właściciel:** każdy kawałek stanu ma jedno źródło prawdy; kopie = desynchronizacja.
4. **Modeluj stany jawnie:** discriminated unions zamiast zup booleanów ([TS: interfejsy i typy](../06-typescript/02-interfejsy-i-typy.md)).
5. **Maszyny stanów** dla złożonych przepływów (checkout, uploader, odtwarzacz): jawne stany i przejścia (XState) zamiast dziesięciu flag.
6. **Granice modułów:** stan featury w jej module, nie w globalnym worku ([struktura projektu](../08-architektura-aplikacji/04-struktura-projektu.md)).

## Pułapki i częste błędy

- Dane z API w globalnym store ręcznie (loading/error flags, ręczna inwalidacja) — przepisywanie query-cache na piechotę.
- Wszystko globalne „bo może się przyda" — sprzężenie każdego z każdym; refaktor graniczy z cudem.
- Duplikowanie stanu (lista + wybrany element jako kopia obiektu zamiast id).
- Stan pochodny w store (filteredList obok list+filter).
- Kontekst o częstej zmienności — lawiny renderów.
- Brak strategii dla stanu URL — filtry znikają po odświeżeniu; „nie da się podlinkować".

## Pytania rekrutacyjne

1. **Jak decydujesz, gdzie żyje stan?** — taksonomia (lokalny/współdzielony/server/URL) + kolokacja; to pytanie-esencja tematu.
2. **Czym server state różni się od client state?** — własność danych, cache, świeżość, rewalidacja; dlaczego dedykowane biblioteki.
3. **Opisz wzorzec Flux/Redux i jego zalety.** — jednokierunkowy przepływ, czyste reducery, przewidywalność; kiedy overkill.
4. **Do czego służy kontekst i czemu nie zastępuje store'a?** — dystrybucja vs zarządzanie; koszt renderów.
5. **Jak zaprojektujesz stan koszyka w e-commerce?** — id produktów (normalizacja), server state produktów, URL dla widoków, persystencja draftu.

## Dalsza lektura

- [TanStack Query docs: „Does this replace client state?"](https://tanstack.com/query/latest/docs/framework/react/guides/does-this-replace-client-state) — kluczowe rozróżnienie
- [Redux Style Guide](https://redux.js.org/style-guide/) — zasady uniwersalne mimo brandu
- [XState docs: State machines](https://stately.ai/docs)
