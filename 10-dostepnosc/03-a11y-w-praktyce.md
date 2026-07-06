# Dostępność w praktyce: focus, klawiatura, kontrast, testowanie

> **Poziom:** 🟡 średni → 🔴 (zarządzanie focusem)
> **Wymagana wiedza:** [ARIA i semantyka](./02-aria-i-semantyka.md), [Zdarzenia](../05-dom-i-web-api/02-zdarzenia.md)

Warsztat codzienny: rzeczy, które wdrażasz w każdym sprincie. Największa techniczna trudność a11y w aplikacjach dynamicznych to **zarządzanie focusem** — reszta to dyscyplina i testowanie.

## Nawigacja klawiaturą i focus

- **Kolejność Tab = kolejność DOM** — układaj DOM logicznie; nie „naprawiaj" kolejności dodatnim `tabindex`.
- **`tabindex`:** `0` — dodaj do kolejności (custom widget), `-1` — focusowalny tylko programowo (`el.focus()`), `>0` — **nie używaj** (rozjeżdża kolejność).
- **Widoczny focus obowiązkowy:** nie usuwaj outline bez zamiennika; nowocześnie:

```css
:focus-visible { outline: 2px solid var(--focus); outline-offset: 2px; }
/* :focus-visible = fokus z klawiatury; klik myszą nie rysuje ramki */
```

- Skip link („przejdź do treści") na początku strony — pierwszy Tab pozwala ominąć nawigację.
- Custom widget = pełny kontrakt klawiatury z [APG](./02-aria-i-semantyka.md) (strzałki w menu/tabs/listbox, Esc zamyka, Home/End). Wzorzec **roving tabindex** — jeden Tab-stop na widget, strzałki wewnątrz.

## Zarządzanie focusem w aplikacjach dynamicznych

Reguła: **po każdej zmianie kontekstu focus ląduje w przewidywalnym miejscu.**

- **Modal:** otwarcie → focus do środka (nagłówek/pierwszy element); **focus trap** (Tab krąży wewnątrz); Esc zamyka; zamknięcie → **focus wraca na trigger**. Natywny `<dialog>` robi większość ([nowoczesny CSS](../03-css/08-nowoczesny-css.md)).
- **Usunięcie elementu z focusem** (skasowanie wiersza) → focus na sąsiada/kontener, nie w próżnię (body).
- **Nawigacja SPA** → focus na nagłówek nowej strony (`h1` z `tabindex="-1"`) + ogłoszenie live region ([routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)).
- Treść dogrywana async: jeśli to wynik akcji użytkownika — przenieś focus lub ogłoś ([live regions](./02-aria-i-semantyka.md)).

## Kontrast i wizualia

- **Kontrast (WCAG AA):** tekst **4,5:1**; duży tekst (≥24px/≥19px bold) **3:1**; elementy UI i stany (ramki inputów, ikony funkcyjne) **3:1**. Sprawdzaj w DevTools (picker pokazuje ratio) lub WebAIM Contrast Checker; projektuj tokenami z zapasem ([zmienne CSS](../03-css/08-nowoczesny-css.md)).
- **Kolor nie może być jedynym nośnikiem** informacji (błąd = kolor + ikona + tekst; wykresy: wzory/etykiety).
- **Rozmiary:** tekst w `rem` (respektuje ustawienia — [jednostki](../03-css/02-box-model-i-jednostki.md)); layout przeżywa zoom 200–400% ([responsywność](../03-css/06-responsywnosc.md)); cele dotykowe ≥24px (WCAG 2.2) / ~44px (komfort).
- **Ruch:** `prefers-reduced-motion` ([animacje](../03-css/07-animacje-i-transformacje.md)); autoplay karuzel — z pauzą.

## Formularze — najczęstsze pole bitwy

Fundament w [formularzach](../02-html/03-formularze.md) (label, natywna walidacja); warstwa a11y:

```html
<label for="email">E-mail</label>
<input id="email" aria-describedby="email-err" aria-invalid="true" autocomplete="email" />
<p id="email-err" role="alert">Podaj adres z „@".</p>
```

Błędy: tekstem przy polu (nie sam kolor ramki), powiązane `aria-describedby`, `aria-invalid`; przy submit z błędami — focus na pierwsze błędne pole lub podsumowanie błędów z linkami.

## Testowanie a11y — piramida

1. **Automaty (łapią ~30–40%):** axe-core — w [DevTools (Lighthouse)](../14-narzedzia/05-devtools.md), w [testach komponentów](../13-testowanie/03-testy-komponentow-i-integracyjne.md) (jest-axe/vitest-axe), w [e2e](../13-testowanie/04-testy-e2e-i-specjalistyczne.md) (@axe-core/playwright), ESLint (eslint-plugin-jsx-a11y).
2. **Klawiatura (5 minut, wykrywa najwięcej):** przejdź flow Tabem — wszystko osiągalne? focus widoczny? kolejność logiczna? Esc działa? nie ma pułapek?
3. **Czytnik ekranu:** NVDA (Windows, darmowy) / VoiceOver (Cmd+F5) — kluczowe ścieżki: czy wszystko ma nazwę, czy zmiany są ogłaszane.
4. **Zoom 400%, tryb wysokiego kontrastu, sam kolor** — szybkie sanity checks.
5. Najlepiej: testy z udziałem użytkowników technologii asystujących (dojrzałe organizacje).

W procesie: kryteria a11y w definition of done, automaty w [CI](../16-devops-dla-frontendu/01-ci-cd.md), audyt manualny przy dużych zmianach.

## Pułapki i częste błędy

- `outline: none` globalnie „bo brzydkie" — bez zamiennika to wykluczenie klawiatury.
- Modal bez trapa i bez powrotu focusa — czytnik „wypada" do tła.
- Dodatni tabindex i „naprawianie" kolejności zamiast poprawy DOM.
- Placeholder jako label; błędy tylko kolorem.
- Testowanie a11y raz na kwartał zamiast w każdym PR.
- Poleganie na `disabled` dla przycisków submit — element znika z Tab i nie tłumaczy czemu; rozważ `aria-disabled` + komunikat.

## Pytania rekrutacyjne

1. **Jak zaimplementujesz dostępny modal?** — focus in/trap/return, Esc, aria-modal/role=dialog albo `<dialog>`; tło inert.
2. **Co się dzieje z focusem po nawigacji w SPA i co z tym robisz?** — nic domyślnie; focus na h1 + live region.
3. **Jakie są wymogi kontrastu WCAG AA?** — 4,5:1 / 3:1 duży tekst / 3:1 UI.
4. **Jak testujesz dostępność?** — piramida: axe w CI, klawiatura, czytnik, zoom; ograniczenia automatów.
5. **Czym jest roving tabindex?** — jeden Tab-stop, strzałki wewnątrz widgetu; APG.

## Dalsza lektura

- [web.dev: Learn Accessibility — Focus](https://web.dev/learn/accessibility/focus)
- [WebAIM: Keyboard Accessibility](https://webaim.org/techniques/keyboard/) i [Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Deque University: axe rules](https://dequeuniversity.com/rules/axe/)
