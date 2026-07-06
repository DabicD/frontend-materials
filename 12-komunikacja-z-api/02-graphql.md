# GraphQL

> **Poziom:** 🟡 średni → 🔴 (rozważania architektoniczne)
> **Wymagana wiedza:** [REST](./01-rest.md), [Fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)

GraphQL to język zapytań do API i runtime, w którym **klient deklaruje, jakich danych potrzebuje**, a serwer zwraca dokładnie to — nie więcej, nie mniej. Rozwiązuje konkretne bóle REST (over/under-fetching, wodospady), wprowadzając własne kompromisy. Nie musisz go kochać — musisz wiedzieć, kiedy pasuje.

## Problem, który rozwiązuje

REST: sztywne endpointy zwracają ustalony kształt.
- **Over-fetching** — `/users/42` zwraca 30 pól, potrzebujesz 2.
- **Under-fetching / N+1** — by złożyć widok, wołasz `/user`, potem `/user/orders`, potem `/order/items`… wodospad requestów ([REST — N+1](./01-rest.md)).

GraphQL: **jeden endpoint** (`POST /graphql`), jedno zapytanie o dokładnie potrzebny graf danych.

## Podstawy: schema, query, mutation

```graphql
# Schema (kontrakt, silnie typowany — po stronie serwera)
type User { id: ID!, name: String!, orders(last: Int): [Order!]! }
type Order { id: ID!, total: Float!, items: [Item!]! }

# Query — odczyt; klient wybiera pola i zagnieżdżenia
query UserDashboard($id: ID!) {
  user(id: $id) {
    name
    orders(last: 5) { total items { name } }   # wszystko w JEDNYM requeście
  }
}

# Mutation — zapis
mutation AddToCart($productId: ID!) {
  addToCart(productId: $productId) { cart { itemCount } }   # zwraca zaktualizowany stan
}
```

- **Fragments** — reużywalne kawałki selekcji (kolokowane z komponentami: komponent deklaruje swoje potrzeby danych):

```graphql
fragment CartBadge on Cart { itemCount total }
```

- **Subscriptions** — dane w czasie rzeczywistym (zwykle przez WebSocket — [czas rzeczywisty](./03-czas-rzeczywisty.md)).
- **Wszystko silnie typowane** → codegen typów TS z schemy (typy zawsze zgodne z API — [TS w praktyce](../06-typescript/05-typescript-w-praktyce.md)); introspekcja i eksplorator (GraphiQL).

## Cechy specyficzne (i ich konsekwencje dla frontendu)

- **Zwykle `POST`** na jeden URL → standardowy [HTTP cache](../09-wydajnosc/04-strategie-cachowania.md) i CDN **nie działają** out of the box; cache przenosi się do **klienta** (Apollo/urql/Relay z cache znormalizowanym po `id`+`__typename`).
- **Błędy inaczej:** zwykle `200 OK` z tablicą `errors` w body (obok `data`) — nie polegaj na statusie HTTP; obsłuż częściowe wyniki (część pól `data`, część w `errors`).
- **Kształt odpowiedzi = kształt zapytania** — przewidywalny, ale klient odpowiada za sensowną selekcję.

## Klienci GraphQL

Rzadko woła się GraphQL gołym `fetch` — biblioteki dają cache, dedup, normalizację, optymistyczne update'y:
- **Apollo Client** — najbogatszy, cache znormalizowany, duży ekosystem.
- **urql** — lżejszy, modularny.
- **Relay** — najbardziej opiniowany/wydajny (fragmenty, paginacja), stroma krzywa; duże aplikacje.
- To one realizują [wzorce pracy z danymi](./04-wzorce-pracy-z-danymi.md) (cache, stany, invalidacja) analogicznie do TanStack Query w świecie REST.

## GraphQL vs REST — kiedy co

| | GraphQL | REST |
|---|---|---|
| Pobieranie danych | dokładnie potrzebne, 1 request | sztywne endpointy, over/under-fetch |
| HTTP cache/CDN | trudne (POST, klient-cache) | natywne (GET+nagłówki) |
| Typowanie/schema | wbudowane + introspekcja | opcjonalnie (OpenAPI) |
| Krzywa wejścia / narzędzia serwera | wyższa | niska |
| Wiele różnych klientów/widoków | świetne (elastyczność) | mnożenie endpointów |
| Proste CRUD / pliki / cache brzegowy | overkill | idealne |

**Heurystyka:** GraphQL błyszczy przy **wielu klientach o różnych potrzebach danych** i złożonych, zagnieżdżonych grafach (aplikacje mobilne + web + partnerzy). Dla prostego CRUD, uploadów i gdy zależy Ci na cache brzegowym — REST jest prostszy. Częsty kompromis: **BFF (Backend for Frontend)** agregujący REST-y pod potrzeby UI daje część korzyści GraphQL bez jego infrastruktury.

## Pułapki i częste błędy

- Oczekiwanie, że HTTP cache „po prostu zadziała" — trzeba cache klienta/persisted queries.
- Ignorowanie tablicy `errors` przy statusie 200 (i częściowych danych).
- Traktowanie GraphQL jako „lepszego REST wszędzie" — koszt: cache, złożoność serwera, rate limiting/koszt zapytań (klient może poprosić o kosztowny graf → potrzeba limitów głębokości/kosztu po stronie serwera).
- Brak codegenu — ręczne typy rozjeżdżają się ze schemą.
- Over-fetching wraca tylnymi drzwiami, gdy komponenty proszą „na zapas" — dyscyplina fragmentów.

## Pytania rekrutacyjne

1. **Jaki problem REST rozwiązuje GraphQL?** — over/under-fetching, N+1; jeden request o dokładny graf.
2. **Czemu cache w GraphQL jest trudniejszy?** — POST/jeden URL; cache przenosi się do klienta (normalizacja po id).
3. **Jak GraphQL sygnalizuje błędy?** — 200 + `errors` (+ częściowe `data`); nie polegaj na statusie.
4. **Kiedy wybierzesz REST zamiast GraphQL?** — proste CRUD, cache brzegowy, upload, mały zespół; BFF jako środek.
5. **Co dają fragmenty i codegen?** — kolokacja potrzeb danych z komponentem; typy zsynchronizowane ze schemą.

## Dalsza lektura

- [Oficjalny: graphql.org/learn](https://graphql.org/learn/)
- [How to GraphQL](https://www.howtographql.com/)
- [Apollo Client docs](https://www.apollographql.com/docs/react/) / [urql](https://commerce.nearform.com/open-source/urql/)
