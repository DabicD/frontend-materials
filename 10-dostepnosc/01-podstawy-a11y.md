# Podstawy dostępności (a11y)

> **Poziom:** 🟢 podstawowy (obowiązkowy)
> **Wymagana wiedza:** [Semantyka HTML](../02-html/02-semantyka.md)

Dostępność (accessibility, a11y) = projektowanie tak, by z produktu mogły korzystać osoby z niepełnosprawnościami — a w praktyce **wszyscy** (sytuacyjne ograniczenia: słońce na ekranie, jedna ręka zajęta, wolne łącze, wiek). Od 2025 r. to także twarde prawo w UE. Dla seniora a11y nie jest „featurą" — jest kryterium jakości, jak wydajność.

## Kogo dotyczy — szersze spojrzenie

- **Wzrok:** niewidomi (czytniki ekranu), słabowidzący (zoom, kontrast), daltonizm (~8% mężczyzn!).
- **Ruch:** nawigacja klawiaturą/przełącznikami, drżenie rąk (małe cele dotykowe).
- **Słuch:** napisy dla treści audio/wideo.
- **Poznawcze:** prosty język, przewidywalne wzorce, brak migotania, możliwość zatrzymania ruchu.
- **Sytuacyjne/tymczasowe:** kontuzja, hałas, stary telefon — dostępność poprawia produkt dla wszystkich (krzywa jak podjazdy dla wózków, z których korzystają wszyscy z walizkami).

## Jak korzystają z sieci technologie asystujące

- **Czytnik ekranu** (NVDA — darmowy/Windows, VoiceOver — macOS/iOS, JAWS, TalkBack) czyta **accessibility tree** — strukturę zbudowaną z DOM + semantyki + ARIA ([szczegóły](./02-aria-i-semantyka.md)). Nawigacja: po nagłówkach, landmarkach, linkach, formularzach — nie liniowo!
- **Klawiatura:** Tab/Shift+Tab między elementami interaktywnymi, Enter/Spacja aktywacja, strzałki w widgetach. Jeśli coś nie działa z klawiatury — nie działa też z wieloma innymi technologiami.
- Zoom do 400%, tryby wysokiego kontrastu, sterowanie głosem, wyświetlacze brajlowskie.

**Minimum praktyczne:** przejdź swój flow samą klawiaturą i raz w życiu włącz czytnik (NVDA/VoiceOver) — pół godziny, która zmienia perspektywę na zawsze.

## WCAG — standard

**WCAG (Web Content Accessibility Guidelines)** organizuje wymagania wokół zasad **POUR**:

- **P**erceivable (postrzegalność) — alternatywy tekstowe, napisy, kontrast, responsywność do 400% zoom.
- **O**perable (funkcjonalność) — klawiatura, czas na reakcję, brak pułapek focusa, brak treści wywołujących ataki (migotanie).
- **U**nderstandable (zrozumiałość) — czytelny język, przewidywalna nawigacja, pomoc przy błędach ([formularze](../02-html/03-formularze.md)).
- **R**obust (solidność) — poprawny kod działający z technologiami asystującymi.

**Poziomy zgodności:** A (minimum) → **AA (standard prawny i rynkowy — celuj tu)** → AAA (wybrane konteksty). Wersje: 2.1/2.2 (2.2 dodała m.in. minimalne rozmiary celów, widoczność focusa). Kryteria sukcesu są testowalne — np. kontrast tekstu **4,5:1** (AA).

## Prawo (stan na 2026)

- **European Accessibility Act (EAA)** — od **28 czerwca 2025** obowiązuje dla e-commerce, bankowości, transportu, e-booków itd. w całej UE (w Polsce: ustawa o zapewnianiu spełniania wymagań dostępności niektórych produktów i usług). Techniczny punkt odniesienia: norma **EN 301 549** ≈ WCAG 2.1 AA.
- Sektor publiczny: dyrektywa + ustawa o dostępności cyfrowej (od 2019).
- USA: ADA (procesy sądowe), Section 508.
- Konsekwencja: pytania o a11y na rozmowach przestały być teoretyczne — to ryzyko prawne i biznesowe firm.

## Business case (na rozmowę i dla zespołu)

1. Rynek: ~16% populacji z niepełnosprawnościami + starzejące się społeczeństwa.
2. Prawo: EAA/ADA — kary i pozwy.
3. Jakość: dostępny kod = semantyczny = lepsze [SEO](../02-html/05-seo-i-metadane.md), łatwiejsze [testy](../13-testowanie/03-testy-komponentow-i-integracyjne.md) (selektory po rolach!), mniej bugów UX.
4. Koszt: a11y projektowana od początku ≈ tania; doklejana po audycie ≈ droga. Dlatego należy do definition of done, nie do backlogu „kiedyś".

## Pułapki i częste błędy (myślowe)

- „Nasi użytkownicy nie są niepełnosprawni" — nie wiesz tego (niewidoczne niepełnosprawności, sytuacyjne); a prawo nie pyta.
- A11y = „dodamy ARIA na końcu" — ARIA to ostatnia deska, fundamentem jest [semantyka](../02-html/02-semantyka.md).
- Overlay'e/„accessibility widgets" obiecujące zgodność jednym skryptem — nie działają, bywają pozywane.
- Testowanie tylko automatem — łapie ~30–40% problemów ([praktyka](./03-a11y-w-praktyce.md)).
- Dostępność jako projekt jednorazowy zamiast procesu (każdy nowy feature może ją zepsuć).

## Pytania rekrutacyjne

1. **Czym jest WCAG i co oznacza poziom AA?** — zasady POUR, testowalne kryteria, AA jako standard prawny.
2. **Jak osoby niewidome korzystają ze stron?** — czytniki, accessibility tree, nawigacja po nagłówkach/landmarkach.
3. **Jakie wymogi prawne dotyczą dostępności w UE?** — EAA od 2025, EN 301 549/WCAG 2.1 AA; kogo obejmuje.
4. **Jak uzasadnisz biznesowo inwestycję w a11y?** — rynek, prawo, jakość, koszt wczesny vs późny.
5. **Od czego zaczniesz poprawę dostępności istniejącego produktu?** — audyt (auto+manualny), krytyczne ścieżki, proces w DoD — [praktyka](./03-a11y-w-praktyce.md).

## Dalsza lektura

- [web.dev: Learn Accessibility](https://web.dev/learn/accessibility)
- [WCAG 2.2 (W3C)](https://www.w3.org/TR/WCAG22/) + [How to Meet WCAG (quick ref)](https://www.w3.org/WAI/WCAG22/quickref/)
- [WebAIM — artykuły i badania (Screen Reader Survey, Million)](https://webaim.org/)
