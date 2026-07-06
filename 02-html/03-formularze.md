# Formularze

> **Poziom:** 🟢 podstawowy → 🟡 średni (walidacja, UX)
> **Wymagana wiedza:** [Semantyka](./02-semantyka.md)

Formularze to najczęstszy punkt styku użytkownika z logiką aplikacji — i miejsce, gdzie frontend najczęściej „wynajduje koło" gorzej, niż zrobiła to przeglądarka. Natywne mechanizmy (typy inputów, walidacja, autofill) dają za darmo UX, dostępność i obsługę mobile.

## Fundament: `<form>`, `<label>`, `name`

```html
<form action="/subscribe" method="post">
  <label for="email">E-mail</label>
  <input id="email" name="email" type="email" required autocomplete="email" />
  <button type="submit">Zapisz się</button>
</form>
```

- **Każdy input ma `<label>`** — powiązany przez `for`/`id` (lub owinięcie). Label = klikalne pole + nazwa dla czytnika ekranu. Placeholder **nie zastępuje** labela (znika przy pisaniu, słaby kontrast).
- **`name`** — klucz, pod którym wartość trafia do danych formularza; bez `name` pole nie istnieje dla submita.
- **`<button type>`**: w formularzu domyślny typ to **`submit`**! Przycisk „pomocniczy" bez `type="button"` będzie wysyłał formularz — klasyczny bug.
- Submit przez **Enter** działa za darmo, jeśli używasz `<form>` + submit button. Ręczne `onClick` na divach to gubi.
- W JS: zdarzenie `submit` na formularzu (nie `click` na przycisku!), `event.preventDefault()` dla obsługi własnej, dane przez `FormData` ([fetch i FormData](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).

## Typy inputów — używaj właściwych

`email`, `number`, `tel`, `url`, `date`, `time`, `password`, `search`, `checkbox`, `radio`, `file`, `range`, `color`, `hidden`…

Właściwy typ daje: **klawiaturę mobilną** dopasowaną do treści, natywne kontrolki (date picker), wstępną walidację. Dodatkowo `inputmode` (np. `inputmode="numeric"` dla kodów PIN — lepsze niż `type="number"` dla identyfikatorów) i `enterkeyhint`.

Pozostałe elementy: `<select>`, `<textarea>`, `<fieldset>`+`<legend>` (grupowanie — obowiązkowe dla grup radio/checkbox), `<datalist>` (autouzupełnianie sugestii), `<output>`.

## Walidacja natywna (Constraint Validation API)

Atrybuty deklaratywne: `required`, `minlength`/`maxlength`, `min`/`max`/`step`, `pattern`, `type` (email/url).

```html
<input name="nick" required minlength="3" pattern="[a-z0-9]+" />
```

- Przeglądarka blokuje submit i pokazuje dymek z komunikatem. Pseudoklasy CSS: `:valid`, `:invalid`, `:user-invalid` (dopiero po interakcji — lepszy UX niż `:invalid` od startu).
- API w JS: `input.checkValidity()`, `input.validity` (obiekt z flagami: `valueMissing`, `tooShort`…), `input.setCustomValidity('komunikat')`, `form.reportValidity()`.
- `novalidate` na formularzu wyłącza natywne dymki, ale zostawia API — typowy setup przy własnym UI błędów: walidujesz przez `checkValidity()`, komunikaty renderujesz sam (z `aria-describedby` i `aria-invalid` — [a11y w praktyce](../10-dostepnosc/03-a11y-w-praktyce.md)).

**Zawsze waliduj też na backendzie** — walidacja frontendowa to UX, nie [bezpieczeństwo](../11-bezpieczenstwo/01-ataki-na-frontend.md) (każdy request można wysłać z pominięciem przeglądarki).

## Autofill i `autocomplete`

Atrybut `autocomplete` podpowiada przeglądarce/menedżerowi haseł znaczenie pola: `email`, `name`, `current-password`, `new-password`, `one-time-code`, `cc-number`, `street-address`…

- Dobrze opisany formularz wypełnia się jednym tapnięciem — ogromny wpływ na konwersję na mobile.
- `autocomplete="new-password"` podpowiada generowanie hasła; `one-time-code` auto-wkleja SMS-owe kody.
- `autocomplete="off"` jest w większości przeglądarek **ignorowane** dla pól logowania (celowo).

## UX formularzy — zestaw zasad senior-level

1. Waliduj **przy opuszczeniu pola lub przy submit** — nie przy każdym znaku (chyba że pole już było błędne — wtedy waliduj na bieżąco, by pokazać naprawę: wzorzec „reward early, punish late").
2. Komunikat błędu **przy polu**, konkretny („Podaj adres z @"), nie generyczny („Błąd").
3. Nie czyść formularza po błędzie; zachowaj wartości przy powrocie.
4. Disabled submit button podczas wysyłki + stan ładowania; obsłuż podwójny klik.
5. Stany asynchroniczne (błąd sieci, błędy z API per-pole) zaplanuj z góry — wzorce: [praca z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md).

## Pułapki i częste błędy

- `<button>` bez `type="button"` w formularzu — niechciany submit.
- Placeholder zamiast label — dostępność i UX leżą.
- `type="number"` dla numeru telefonu/karty — ucina zera wiodące, pozwala na `e`, kręci scrollem wartość; użyj `type="tel"`/`inputmode`.
- Blokowanie wklejania do pól (haseł!) — walka z menedżerami haseł.
- Własne selecty/checkboxy z divów bez odtworzenia klawiatury i ARIA — wielki koszt, mały zysk.
- Walidacja tylko na frontendzie.

## Pytania rekrutacyjne

1. **Dlaczego label jest ważny i jak go wiązać z inputem?** — dostępność (nazwa pola), większy obszar kliknięcia; `for`/`id` lub zagnieżdżenie.
2. **Jak działa natywna walidacja i jak ją ucywilizować pod własny design?** — atrybuty + Constraint Validation API; `novalidate` + `checkValidity()` + własne komunikaty z ARIA.
3. **Czemu walidacja kliencka nie wystarcza?** — request można spreparować; frontend = UX, backend = bezpieczeństwo/integralność.
4. **Co daje atrybut `autocomplete`?** — poprawny autofill (konwersja, mobile), wsparcie menedżerów haseł, OTP z SMS.
5. **Domyślny `type` przycisku w formularzu?** — `submit`; stąd obowiązkowe `type="button"` dla akcji pomocniczych.

## Dalsza lektura

- [MDN: Web forms — kompletny kurs](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms)
- [web.dev: Learn Forms](https://web.dev/learn/forms)
- [MDN: Constraint validation](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation)
