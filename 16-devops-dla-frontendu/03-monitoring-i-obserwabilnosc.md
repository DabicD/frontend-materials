# Monitoring i obserwowalność

> **Poziom:** 🟡 średni → 🔴 (kultura obserwowalności)
> **Wymagana wiedza:** [Obsługa błędów](../04-javascript/11-obsluga-bledow.md), [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md), [CI/CD](./01-ci-cd.md)

Deploy to nie koniec — to początek życia kodu w rękach użytkowników. **Bez obserwowalności jesteś ślepy:** nie wiesz, że aplikacja się psuje, dopóki ktoś nie zadzwoni. Dojrzały frontend jest instrumentowany: błędy, wydajność i zachowania są mierzone w produkcji. To odróżnia „wrzuciłem i mam nadzieję" od profesjonalnej eksploatacji.

## Dlaczego to sprawa frontendu

Środowisko użytkownika jest **niekontrolowane**: tysiące kombinacji przeglądarek, urządzeń, sieci, rozszerzeń, wersji. Błąd widoczny tylko na starym Androidzie przez wolną sieć **nie pojawi się** w Twoim [lab/CI](./01-ci-cd.md) — tylko w [field/RUM](../09-wydajnosc/01-core-web-vitals.md). Produkcja to jedyne prawdziwe środowisko testowe dla tej różnorodności.

## Trzy filary (i frontendowa rzeczywistość)

Klasyczne filary obserwowalności: **logi**, **metryki**, **traces**. We frontendzie przekładają się na:

1. **Error tracking** — przechwytywanie i agregacja błędów.
2. **RUM (Real User Monitoring)** — wydajność i metryki z prawdziwych sesji.
3. **Analytics/product** — zachowania użytkowników (osobny cel, ale wspólna infrastruktura).

## Error tracking

Automatyczne łapanie i raportowanie błędów runtime (Sentry, Bugsnag, Rollbar…):

```js
window.addEventListener('error', e => report(e.error));
window.addEventListener('unhandledrejection', e => report(e.reason));   // [asynchroniczność]
```

Co dobre narzędzie daje ponad goły log:
- **Grupowanie** podobnych błędów (fingerprinting) — 10 000 wystąpień → 1 issue z liczbą.
- **Kontekst:** [stack trace](../04-javascript/11-obsluga-bledow.md) zdemapowany [source mapami](../14-narzedzia/03-bundlery-i-proces-budowania.md) (czytelny, nie zminifikowany!), przeglądarka/OS/urządzenie, wersja aplikacji (release), user id (zanonimizowany).
- **Breadcrumbs** — ślad akcji przed błędem (kliknięcia, nawigacje, requesty) — reprodukcja bez pytania użytkownika.
- **Session replay** — nagranie sesji prowadzącej do błędu (uwaga na [prywatność](../11-bezpieczenstwo/01-ataki-na-frontend.md) — maskuj dane wrażliwe).
- Integracja z [error boundaries](../07-frameworki-koncepcyjnie/02-model-komponentowy.md) frameworka.

Powiązanie z **release** (wersją buildu) pozwala wykryć „nowy deploy = skok błędów" → szybki [rollback](./01-ci-cd.md).

## RUM — wydajność u prawdziwych użytkowników

Pomiar [Core Web Vitals](../09-wydajnosc/01-core-web-vitals.md) i własnych metryk w produkcji ([web-vitals](../09-wydajnosc/01-core-web-vitals.md)):

```js
import { onLCP, onINP, onCLS } from 'web-vitals';
[onLCP, onINP, onCLS].forEach(fn => fn(m => beacon('/rum', m)));
```

- Patrz na **p75/percentyle**, segmentuj (urządzenie, kraj, typ strony, [połączenie](../01-fundamenty-sieci/01-jak-dziala-internet.md)) — średnia kłamie.
- Własne metryki biznesowe: „czas do interaktywnej listy", „czas do pierwszego wyniku wyszukiwania" (`performance.mark/measure`).
- Wysyłka: `navigator.sendBeacon`/fetch keepalive na `visibilitychange` (przeżywa zamknięcie karty — [przydatne API](../05-dom-i-web-api/07-przydatne-api-przegladarki.md)).

## Alerty i SLO

- **Alerty** na progach: skok error rate, LCP p75 > budżet, spadek konwersji, wzrost `5xx` z [API](../12-komunikacja-z-api/01-rest.md). Alert ma być **akcjonowalny** — zmęczenie alertami (za dużo szumu) jest gorsze niż ich brak.
- **SLO/SLI** (Service Level Objectives/Indicators) — mierzalne cele („99,9% sesji bez błędu krytycznego", „LCP p75 < 2,5 s dla 90% wizyt"); error budget jako język rozmowy o ryzyku wdrożeń.
- Powiązanie z [budżetami wydajności](./01-ci-cd.md) w CI — łap regresje przed produkcją, monitoruj po.

## Product analytics i prywatność

- Zdarzenia produktowe (funnel, retencja, A/B) — [feature flags](../15-inzynieria-oprogramowania/03-praca-z-legacy.md) często zintegrowane z eksperymentami.
- **Prywatność jako wymóg, nie dodatek:** RODO/consent, minimalizacja danych, anonimizacja, maskowanie w session replay, respektowanie zgód. Skrypty analityczne to też [waga](../09-wydajnosc/02-optymalizacja-ladowania.md) i [ryzyko third-party](../11-bezpieczenstwo/01-ataki-na-frontend.md) — audytuj.

## Pętla obserwowalności (kultura)

```
deploy → monitoruj (błędy/RUM/metryki) → wykryj anomalię/alert → diagnozuj (breadcrumbs/replay/trace)
       → napraw/rollback → zweryfikuj w produkcji → wyciągnij wnioski (blameless postmortem)
```

Dojrzały zespół zamyka tę pętlę: incydenty prowadzą do **blameless postmortem** (co zawiodło w systemie, nie kto zawinił) i usprawnień — nie do szukania winnych.

## Pułapki i częste błędy

- Deploy bez monitoringu — dowiadujesz się o awarii od użytkowników (albo nie dowiadujesz wcale).
- Brak source maps w error trackingu — nieczytelne, zminifikowane stack trace.
- Patrzenie na średnią zamiast percentyli/segmentów — maskuje ból mniejszości (która i tak odchodzi).
- Zbyt wiele alertów (szum) → ignorowanie wszystkich, w tym prawdziwych.
- Logowanie/replay danych wrażliwych (tokeny, PII) — incydent prywatności.
- Traktowanie „zielonego CI" jako gwarancji — CI nie widzi środowisk użytkowników.
- Instrumentacja obciążająca wydajność, którą ma mierzić (ciężkie skrypty RUM/analytics).

## Pytania rekrutacyjne

1. **Skąd wiesz, że produkcja działa dobrze?** — error tracking + RUM + alerty/SLO; nie „brak zgłoszeń".
2. **Co dobre narzędzie do błędów daje ponad `console.log`?** — grupowanie, kontekst, source maps, breadcrumbs, release.
3. **Jak mierzysz wydajność u prawdziwych użytkowników?** — RUM (web-vitals), percentyle, segmentacja, sendBeacon.
4. **Czemu CI nie wystarcza i potrzebny monitoring?** — nie odwzoruje różnorodności środowisk użytkowników.
5. **Jak podejść do prywatności w monitoringu?** — consent, minimalizacja, anonimizacja, maskowanie replay.
6. **Co robisz, gdy alert pokazuje skok błędów po deployu?** — diagnoza (release/breadcrumbs) → rollback/feature flag → postmortem.

## Dalsza lektura

- [Sentry — dokumentacja (frontend monitoring)](https://docs.sentry.io/)
- [web.dev: Monitor performance in the field (RUM)](https://web.dev/articles/vitals-measurement-getting-started)
- [Google SRE Book: SLO / Postmortem culture](https://sre.google/books/)
