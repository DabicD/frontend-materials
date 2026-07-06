# Wzorce renderowania: CSR, SSR, SSG, ISR

> **Poziom:** 🔴 zaawansowany (kluczowy temat architektoniczny)
> **Wymagana wiedza:** [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md), [Po co frameworki](../07-frameworki-koncepcyjnie/01-po-co-frameworki.md), [SEO](../02-html/05-seo-i-metadane.md)

„Gdzie i kiedy generujemy HTML?" — to jedna z najważniejszych decyzji architektonicznych frontendu. Odpowiedź wpływa na: wydajność startu, SEO, koszty infrastruktury i złożoność. Meta-frameworki (Next/Nuxt/SvelteKit/Astro) istnieją głównie po to, by dać te wzorce out of the box.

## Słownik wzorców

**CSR (Client-Side Rendering)** — serwer wysyła pusty HTML + bundle JS; przeglądarka renderuje wszystko:

```
serwer: <div id="root"></div> + app.js  →  klient: pobierz JS → wykonaj → render → fetch danych → render
```

- ✚ najprostszy hosting (statyki), bogata interaktywność, tania infrastruktura.
- ✖ wolny start (łańcuch: HTML→JS→dane), pusty HTML dla botów ([SEO](../02-html/05-seo-i-metadane.md)), słabe urządzenia cierpią.
- Dobre dla: aplikacji za loginem (dashboardy, panele), gdzie SEO nieistotne.

**SSR (Server-Side Rendering)** — HTML generowany **na żądanie** per request:

- ✚ szybki first paint (treść w pierwszej odpowiedzi), pełny HTML dla botów, dane osobiste per user.
- ✖ serwer aplikacyjny (koszty, skalowanie), TTFB zależny od backendu, złożoność (kod uniwersalny: serwer+klient).
- Dobre dla: e-commerce, treści dynamiczne + SEO.

**SSG (Static Site Generation)** — HTML generowany **w build time**:

- ✚ najszybsze możliwe serwowanie (czysta statyka z [CDN](../01-fundamenty-sieci/04-domeny-hosting-cdn.md)), tanie, bezpieczne.
- ✖ dane zamrożone do następnego builda; build 10k stron trwa.
- Dobre dla: blogi, docsy, landing pages, katalogi zmieniane rzadko.

**ISR (Incremental Static Regeneration)** — hybryda: statyka regenerowana **po czasie/na żądanie** (stale-while-revalidate na poziomie stron): świeżość bliska SSR przy koszcie bliskim SSG. Dobre dla: katalogi produktów, treści zmieniane co minuty/godziny.

Do tego **rendering na edge** ([hosting](../16-devops-dla-frontendu/02-hosting-i-deployment.md)) — SSR w lokalizacjach CDN: krótszy RTT, ograniczone runtime'y.

## Hydration — cena łączenia światów

SSR/SSG dostarcza HTML, ale interaktywność wymaga JS. **Hydracja** = pobranie bundle'a, odtworzenie drzewa komponentów i **podpięcie** listenerów/stanu do istniejącego DOM.

- Problem **uncanny valley:** strona *wygląda* na gotową, ale nie reaguje (JS jeszcze się wykonuje) — mierzone przez INP/TBT ([Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md)).
- **Hydration mismatch** — HTML z serwera ≠ pierwszy render klienta (daty, `Math.random`, treść zależna od `window`) → błędy i przeskoki; renderuj deterministycznie, różnice dopiero po mountcie.
- Skoro klient i tak wykonuje kod komponentów — SSR **nie zmniejsza** JS; poprawia percepcję startu i SEO.

**Odpowiedzi ekosystemu na koszt hydracji:**

- **Streaming SSR** — serwer wysyła HTML kawałkami (shell od razu, wolne sekcje później), klient hydratuje selektywnie/priorytetowo.
- **Islands architecture** (Astro) — strona to statyczny HTML + **wyspy interaktywności**; JS ładuje się tylko dla wysp (`client:visible`). Idealne dla stron treściowych z odrobiną dynamiki.
- **Server Components** (React/Next) — część komponentów wykonuje się **tylko na serwerze** (dostęp do DB, zero ich JS w bundle'u); klienckie komponenty tylko tam, gdzie interakcja. Zmiana modelu myślenia: domyślnie serwer, interaktywność jako wyjątek.
- **Resumability** (Qwik) — zamiast hydracji: serializacja stanu + leniwe wznawianie handlerów; brak kosztu „odtwarzania".

## Jak wybrać — drzewo decyzyjne

1. **Treść publiczna, SEO/social ważne?** Nie → CSR wystarczy. Tak ↓
2. **Jak często zmienia się treść?** Rzadko (build) → SSG; co minuty/godziny → ISR; per request/per user → SSR.
3. **Ile interaktywności?** Punktowa na stronach treściowych → islands; aplikacyjna wszędzie → SSR/CSR z hydracją.
4. **Mieszaj per trasa** — meta-frameworki pozwalają: landing SSG, katalog ISR, koszyk CSR, strona produktu SSR. To standardowa, dojrzała odpowiedź.

## Pułapki i częste błędy

- SPA/CSR dla publicznego e-commerce „bo znamy" — SEO i social snippets leżą ([SEO](../02-html/05-seo-i-metadane.md)).
- SSR „dla wydajności" bez pomiaru — wolny backend = wolny TTFB; czasem SSG/ISR bije SSR o rząd wielkości.
- Hydration mismatch ignorowany jako „warning" — realne bugi i kary wydajnościowe.
- Dane wrażliwe w HTML z SSR (serializowany stan) — trafiają do source i cache; audytuj, co serializujesz.
- Cache'owanie odpowiedzi SSR z danymi usera na CDN ([nagłówki `private`](../01-fundamenty-sieci/02-protokol-http.md)).
- Wybór wzorca raz na zawsze dla całej aplikacji zamiast per trasa.

## Pytania rekrutacyjne

1. **Porównaj CSR/SSR/SSG/ISR — kompromisy.** — tabela wyżej; pokaż myślenie per przypadek.
2. **Czym jest hydracja i jakie ma koszty?** — podpinanie interaktywności; uncanny valley, TBT/INP, mismatch.
3. **Jak działa islands architecture i kiedy ją wybierzesz?** — statyczny HTML + wyspy JS; strony treściowe.
4. **Co dają Server Components?** — komponenty bez JS w bundle'u, dostęp do zasobów serwera; podział server/client.
5. **Zaprojektuj strategię renderowania dla sklepu internetowego.** — per trasa: SSG/ISR katalog, SSR wyszukiwanie/produkt (świeżość), CSR koszyk/konto.

## Dalsza lektura

- [Patterns.dev: Rendering patterns](https://www.patterns.dev/#rendering-patterns)
- [web.dev: Rendering on the Web](https://web.dev/articles/rendering-on-the-web) — klasyka porządkująca pojęcia
- [Astro docs: Islands](https://docs.astro.build/en/concepts/islands/)
