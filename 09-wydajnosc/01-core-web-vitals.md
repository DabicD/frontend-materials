# Core Web Vitals i mierzenie wydajności

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md)

Zasada nr 1 wydajności: **najpierw mierz, potem optymalizuj**. Core Web Vitals (CWV) to ustandaryzowane metryki Google opisujące doświadczenie użytkownika — wpływają na [SEO](../02-html/05-seo-i-metadane.md) i, ważniejsze, korelują z konwersją. Senior zna nie tylko nazwy, ale **co każdą metrykę psuje i naprawia**.

## Trzy Core Web Vitals

**LCP (Largest Contentful Paint)** — czas do wyrenderowania największego elementu w viewporcie (hero image, nagłówek). Cel: **≤ 2,5 s**.

- Rozkład LCP = TTFB + pobranie zasobu + render; napraw wg części: wolny TTFB → [SSG/CDN/cache](../08-architektura-aplikacji/01-wzorce-renderowania.md); zasób → priorytety, formaty, preload ([optymalizacja ładowania](./02-optymalizacja-ladowania.md)); render → render-blocking CSS/JS.
- Klasyczne grzechy: hero z `loading="lazy"`, obraz odkrywany dopiero z CSS/JS, brak `fetchpriority="high"`.

**INP (Interaction to Next Paint)** — responsywność: czas od interakcji (klik, klawisz) do **następnej klatki** po jej obsłużeniu; brana pod uwagę najgorsza/reprezentatywna interakcja wizyty. Cel: **≤ 200 ms**. (Zastąpił FID w 2024.)

- INP = input delay (main thread zajęty) + processing (twoje handlery) + presentation (render).
- Naprawa: krótsze taski, dzielenie pracy i yield ([event loop](../04-javascript/09-event-loop.md)), mniej JS, lżejsze re-rendery ([reaktywność](../07-frameworki-koncepcyjnie/03-modele-reaktywnosci.md)), optymistyczny feedback UI.

**CLS (Cumulative Layout Shift)** — suma nieoczekiwanych przesunięć layoutu. Cel: **≤ 0,1**.

- Sprawcy: obrazy/embedy bez wymiarów ([width/height](../02-html/04-multimedia-i-grafika.md)), wstrzykiwane banery/reklamy, webfonty zmieniające metryki (FOUT), treść dokładana nad istniejącą.
- Naprawa: rezerwuj miejsce (`aspect-ratio`, szkielety o właściwych wymiarach), `font-display` + metryki fallbacku, animuj `transform` zamiast właściwości layoutu ([animacje](../03-css/07-animacje-i-transformacje.md)).

## Metryki pomocnicze (diagnostyczne)

- **TTFB** — czas do pierwszego bajta (serwer+sieć); fundament LCP.
- **FCP** — pierwszy jakikolwiek content.
- **TBT (Total Blocking Time)** — suma blokad main threadu >50 ms; labowy proxy INP.
- Long Tasks / Long Animation Frames (LoAF) — diagnostyka INP.

## Lab vs field — kluczowe rozróżnienie

- **Lab (syntetyczne):** Lighthouse, DevTools — kontrolowane warunki, powtarzalne, do debugowania. **Nie jest prawdą o użytkownikach.**
- **Field (RUM — Real User Monitoring):** dane z prawdziwych sesji — CrUX (publiczne, zasila Google), własny RUM ([monitoring](../16-devops-dla-frontendu/03-monitoring-i-obserwabilnosc.md)). Ocenia się **p75** (75. percentyl), per mobile/desktop.
- Rozjazd lab/field jest normą (Twój laptop ≠ mediana użytkowników); decyzje podejmuj po field, debuguj w lab.

## Narzędzia — warsztat

1. **Lighthouse** (DevTools/PageSpeed Insights) — audyt z rekomendacjami; PSI łączy lab + CrUX.
2. **DevTools Performance panel** — nagranie: flame chart main threadu, long tasks, layout shifts, LCP marker ([warsztat DevTools](../14-narzedzia/05-devtools.md)).
3. **web-vitals (biblioteka)** — pomiar w produkcie:

```js
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(m => sendToAnalytics(m)); onINP(m => sendToAnalytics(m)); onCLS(m => sendToAnalytics(m));
```

4. **Performance API** — `performance.mark/measure` do własnych metryk (czas do interaktywnej listy itd.), PerformanceObserver do zasobów/long tasks.
5. WebPageTest — głęboka analiza (waterfall, filmstrip, porównania).

## Jak pracować z wydajnością (proces seniora)

1. Ustal **budżety wydajności** (np. LCP p75 < 2,5 s, bundle < 200 kB — [pilnowane w CI](../16-devops-dla-frontendu/01-ci-cd.md)).
2. Mierz field (RUM) → wybierz najgorszą metrykę/stronę o największym ruchu.
3. Reprodukuj w lab z throttlingiem (CPU 4×, Slow 4G) → znajdź przyczynę w Performance panelu.
4. Napraw → zweryfikuj w lab → obserwuj field (tygodnie).
5. Regresje łap w CI (Lighthouse CI na PR).

## Pułapki i częste błędy

- Optymalizowanie „na czuja" bez pomiaru — tygodnie pracy w niewłaściwym miejscu.
- Lighthouse 100 na MacBooku po światłowodzie jako „dowód" — testuj z throttlingiem, patrz na field p75 mobile.
- Score Lighthouse jako cel sam w sobie (gaming metryk) zamiast doświadczenia użytkownika.
- Mylenie metryk ładowania (LCP) z runtime (INP) — inne przyczyny, inne leki.
- Jednorazowy „sprint wydajnościowy" bez budżetów i CI — regresja wraca w kwartał.

## Pytania rekrutacyjne

1. **Wymień Core Web Vitals i ich progi.** — LCP 2,5 s / INP 200 ms / CLS 0,1; co mierzą.
2. **Strona ma słaby LCP — jak diagnozujesz?** — rozbij na TTFB/zasób/render; narzędzia; typowe przyczyny.
3. **Czym różni się lab od field i po co oba?** — powtarzalność vs prawda; debug vs decyzje.
4. **Co pogarsza INP i jak naprawiasz?** — długie taski/hydracja/re-rendery; yield, splitting, mniej JS.
5. **Jak zapobiegasz regresjom wydajności?** — budżety + Lighthouse CI + RUM z alertami.

## Dalsza lektura

- [web.dev: Core Web Vitals](https://web.dev/articles/vitals)
- [web.dev: Optymalizacje per metryka (LCP/INP/CLS)](https://web.dev/explore/learn-core-web-vitals)
- [Biblioteka web-vitals](https://github.com/GoogleChrome/web-vitals)
