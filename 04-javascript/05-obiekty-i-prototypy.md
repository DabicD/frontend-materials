# Obiekty i prototypy

> **Poziom:** 🟡 średni → 🔴 zaawansowany (mechanika prototypów)
> **Wymagana wiedza:** [Typy i zmienne](./01-typy-i-zmienne.md), [this](./04-this-i-kontekst.md)

JavaScript ma **dziedziczenie prototypowe**: obiekty dziedziczą bezpośrednio od innych obiektów przez ukryty łańcuch. Klasy (ES6) to czytelna składnia nad tym mechanizmem — nie osobny system. Zrozumienie prototypów tłumaczy, skąd obiekt „ma" metody, których mu nie pisałeś.

## Obiekty — szybki fundament

```js
const user = {
  name: 'Ala',
  'z-myslnikiem': 1,
  [`klucz_${id}`]: 2,          // computed property
  greet() { return `Hej, ${this.name}`; },
  get initials() { return this.name[0]; },   // getter
};

user.name; user['z-myslnikiem'];             // dostęp
delete user.name;
'name' in user;                               // sprawdza też prototypy!
Object.hasOwn(user, 'name');                  // tylko własne właściwości
Object.keys(user); Object.entries(user);      // własne, enumerowalne
```

- Klucze są stringami lub symbolami (liczby są konwertowane).
- Kopiowanie płytkie: `{ ...obj }` / `Object.assign({}, obj)`; głębokie: `structuredClone(obj)`.
- Porównywanie — przez referencję ([typy](./01-typy-i-zmienne.md)).
- Deskryptory właściwości: `writable`, `enumerable`, `configurable` + `Object.defineProperty` — mechanika pod spodem getterów i bibliotek reaktywności.
- `Object.freeze(obj)` — płytkie zamrożenie.

## Łańcuch prototypów

Każdy obiekt ma ukryte pole `[[Prototype]]` (dostęp: `Object.getPrototypeOf(obj)`). Odczyt właściwości: **szukaj na obiekcie → na prototypie → na prototypie prototypu → … → `null`.**

```js
const animal = { eats: true, walk() { return 'idzie'; } };
const rabbit = Object.create(animal);   // rabbit.[[Prototype]] = animal
rabbit.jumps = true;

rabbit.eats;    // true — znalezione na prototypie
rabbit.walk();  // 'idzie' — this = rabbit (this zależy od wywołania!)
```

- **Zapis nie idzie do prototypu** — `rabbit.eats = false` tworzy własną właściwość, przesłaniając odziedziczoną (shadowing).
- `{}` dziedziczy z `Object.prototype` (stąd `toString`, `hasOwnProperty`); tablice z `Array.prototype`; funkcje z `Function.prototype`.
- `__proto__` — historyczny akcesor; w kodzie używaj `Object.getPrototypeOf`/`Object.create`. (Zmiana prototypu żywego obiektu = deoptymalizacja silnika.)

## Funkcje konstruujące i właściwość `prototype`

Dwie różne rzeczy o podobnej nazwie: `[[Prototype]]` (ukryty link obiektu) vs **`prototype`** (zwykła właściwość **funkcji**, używana tylko przez `new`).

```js
function User(name) { this.name = name; }
User.prototype.greet = function () { return `Hej, ${this.name}`; };

const u = new User('Ala');
// new robi 4 rzeczy:
// 1. tworzy pusty obiekt,  2. ustawia jego [[Prototype]] = User.prototype,
// 3. woła User z this = ten obiekt,  4. zwraca go (jeśli konstruktor nie zwrócił obiektu)
```

Metody na `prototype` są **współdzielone** przez wszystkie instancje (jedna funkcja w pamięci) — dlatego tam się je umieszcza(ło).

## Klasy — składnia nad prototypami

```js
class User {
  #password;                       // pole prywatne (prawdziwa prywatność!)
  static count = 0;                // statyczne — na klasie, nie instancji
  constructor(name, password) {
    this.name = name;              // pole instancji
    this.#password = password;
    User.count++;
  }
  greet() { return `Hej, ${this.name}`; }      // → User.prototype.greet
  get masked() { return '*'.repeat(8); }
  static from(json) { return new User(json.name); }   // fabryka
}

class Admin extends User {
  constructor(name, pw) {
    super(name, pw);               // OBOWIĄZKOWE przed użyciem this
  }
  greet() { return `${super.greet()} (admin)`; }   // super = metoda rodzica
}
```

Fakty: klasy są funkcjami (`typeof User === 'function'`); ciało klasy jest zawsze strict mode; klasy **nie są hoistowane** jak deklaracje funkcji (TDZ); `extends` ustawia łańcuch: `admin → Admin.prototype → User.prototype → Object.prototype`.

`instanceof` sprawdza, czy `Klasa.prototype` występuje w łańcuchu obiektu.

## Klasy vs kompozycja

Głębokie hierarchie `extends` szybko sztywnieją (problem „goryla trzymającego banana i całą dżunglę"). We frontendzie dominują **kompozycja i funkcje**: obiekt złożony z mniejszych zachowań, moduły + closures ([wzorce projektowe](./13-wzorce-projektowe-js.md)), a frameworki porzuciły klasy na rzecz funkcji + kompozycji ([model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md)). Klasy pozostają świetne dla: błędów niestandardowych ([obsługa błędów](./11-obsluga-bledow.md)), struktur danych, integracji z API klasowymi.

## Pułapki i częste błędy

- Mylenie `prototype` (właściwość funkcji) z `[[Prototype]]` (link obiektu).
- Wywołanie konstruktora bez `new` — `this` domyślne, właściwości lecą w global/undefined.
- Współdzielony stan mutowalny na prototypie (`prototype.items = []`) — wszystkie instancje piszą do jednej tablicy.
- `for...in` iteruje też po odziedziczonych enumerowalnych — do własnych kluczy `Object.keys`/`Object.hasOwn`.
- Strzałki jako metody klasowe wszędzie „na zapas" — każda instancja dostaje własną kopię funkcji (pamięć) zamiast współdzielonej z prototypu; używaj tam, gdzie faktycznie potrzeba związanego `this`.
- Prototype pollution — złośliwe wstrzyknięcie do `Object.prototype` przez np. naiwny deep-merge ([ataki na frontend](../11-bezpieczenstwo/01-ataki-na-frontend.md)).

## Pytania rekrutacyjne

1. **Jak działa wyszukiwanie właściwości w JS?** — łańcuch prototypów aż do null; zapis tworzy własną właściwość.
2. **Co robi operator `new` krok po kroku?** — 4 kroki (obiekt, prototyp, this, return).
3. **Czym różni się `prototype` od `__proto__`/`[[Prototype]]`?** — właściwość funkcji-konstruktora vs ukryty link każdego obiektu.
4. **Czy klasy w JS to „prawdziwe" klasy?** — cukier nad prototypami; różnice: hoisting/TDZ, strict, ale mechanizm ten sam.
5. **Dziedziczenie czy kompozycja — co preferujesz i czemu?** — kompozycja elastyczniejsza; extends dla płytkich, stabilnych hierarchii.
6. **Jak zrobić obiekt bez prototypu i po co?** — `Object.create(null)`; czysta mapa bez odziedziczonych kluczy (dziś lepiej `Map`).

## Dalsza lektura

- [javascript.info: Prototypes, inheritance](https://javascript.info/prototypes)
- [MDN: Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [MDN: Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
