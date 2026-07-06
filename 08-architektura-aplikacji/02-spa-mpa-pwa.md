# SPA, MPA i PWA (+ Service Worker)

> **Poziom:** 🟡 średni → 🔴 (service worker)
> **Wymagana wiedza:** [Wzorce renderowania](./01-wzorce-renderowania.md), [Przechowywanie danych](../05-dom-i-web-api/04-przechowywanie-danych.md)

Trzy skróty opisujące **architekturę nawigacji i dystrybucji** aplikacji. SPA vs MPA to decyzja „gdzie żyje nawigacja"; PWA to warstwa możliwości „aplikacyjnych" (offline, instalacja, push) — niezależna od tamtej decyzji. Tu też mieszka **Service Worker** (dom tematu).

## SPA vs MPA

**SPA (Single-Page Application):** jeden dokument HTML; nawigacja podmienia widoki w JS ([routing kliencki](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)); dane przez API ([fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)).

**MPA (Multi-Page Application):** każda nawigacja = nowy dokument z serwera (klasyczne strony, server-rendered aplikacje, Astro/Rails/htmx).

| | SPA | MPA |
|---|---|---|
| Nawigacja | natychmiastowa po starcie, stan trwa między widokami | pełne przeładowanie (dziś łagodzone: bfcache, speculation rules, View Transitions) |
| Start | cięższy (bundle, hydracja/CSR) | lekki per strona |
| Stan między widokami | naturalny (pamięć) | wymaga serwera/storage |
| Złożoność | duża (routing, stan, [wszystko co przeglądarka robiła sama](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)) | mniejsza; logika na serwerze |
| Typowy przypadek | aplikacje: edytory, dashboardy, komunikatory | treść: sklepy-katalogi, media, docs, marketing |

Wybór nie jest religijny: **jak bardzo „aplikacją"** jest produkt? Ciągła manipulacja współdzielonym stanem (Figma, Gmail) → SPA. Przeglądanie treści → MPA/islands zwykle szybsze i prostsze. Hybrydy (SSR z kliencką nawigacją po hydracji) zacierają granicę.

## PWA (Progressive Web App)

Zestaw technologii czyniących stronę „instalowalną" i działającą offline:

1. **HTTPS** (warunek wejścia),
2. **Web App Manifest** — `manifest.json`: nazwa, ikony, kolory, `display: standalone` (bez UI przeglądarki), instalacja na ekranie głównym,
3. **Service Worker** — mózg offline (niżej).

Do tego API „aplikacyjne": push notifications, Background Sync, File System Access, skróty… ([przydatne API](../05-dom-i-web-api/07-przydatne-api-przegladarki.md)). PWA vs natywna aplikacja: jeden kod, brak sklepów i prowizji, linkowalność; słabsze API na iOS i mniejsza integracja z systemem.

## Service Worker — programowalne proxy sieciowe (dom tematu)

SW to skrypt działający **niezależnie od strony** (osobny wątek, własny cykl życia), który **przechwytuje requesty** swojego origin/scope:

```js
// rejestracja (ze strony)
navigator.serviceWorker.register('/sw.js');

// sw.js — cykl życia: install → waiting → activate → (fetch…)
self.addEventListener('install', (e) => {
  e.waitUntil(caches.open('v1').then(c => c.addAll(['/', '/app.css', '/app.js'])));
});
self.addEventListener('activate', (e) => { /* czyszczenie starych cache */ });
self.addEventListener('fetch', (e) => {
  e.respondWith(
    caches.match(e.request).then(hit => hit ?? fetch(e.request))   // cache-first
  );
});
```

**Fakty o cyklu życia (źródło większości WTF):**
- Nowa wersja SW instaluje się, ale **czeka** (waiting), aż zamkną się wszystkie karty starej — chyba że `skipWaiting()`. Stąd „użytkownicy widzą starą wersję mimo deploya".
- SW startuje i umiera na żądanie — **żaden stan w zmiennych nie przetrwa**; trwałość → [Cache API / IndexedDB](../05-dom-i-web-api/04-przechowywanie-danych.md).
- Działa tylko na HTTPS; scope ogranicza ścieżki.

**Strategie cache (nazwy-standardy — szczegóły i dobór: [strategie cache'owania](../09-wydajnosc/04-strategie-cachowania.md)):**
- **Cache First** — statyki wersjonowane.
- **Network First** (z fallbackiem do cache) — treść, która ma być świeża, ale działać offline.
- **Stale-While-Revalidate** — pokaż z cache, w tle odśwież.
- Offline fallback — dedykowana strona/dane awaryjne.

W praktyce nie pisze się tego ręcznie: **Workbox** (biblioteka Google) generuje SW ze strategiami i precache manifestem zintegrowanym z buildem.

**Pozostałe moce SW:** push notifications (odbiór w tle), Background Sync (ponów mutacje po odzyskaniu sieci — kolejka w IndexedDB), okresowa synchronizacja.

## Pułapki i częste błędy

- SW cache'ujący **HTML i własny plik sw.js** zbyt agresywnie — aplikacja „przykleja się" do starej wersji na zawsze; HTML: network-first/no-cache.
- Brak strategii aktualizacji SW (skipWaiting/clientsClaim/„odśwież, żeby zaktualizować" toast) — użytkownicy w wersji sprzed tygodni.
- PWA jako checkbox (manifest + pusty SW dla „installable") bez przemyślanego offline — sama złożoność, zero wartości.
- Zapomniany SW ze starego projektu na tej samej domenie — serwuje trupa; umiej go wyrejestrować (`registration.unregister()`, DevTools → Application).
- Traktowanie SPA jako jedynego „profesjonalnego" wyboru — patrz tabela.

## Pytania rekrutacyjne

1. **SPA vs MPA — jak wybierasz?** — charakter produktu (aplikacja vs treść), stan, SEO, złożoność; hybrydy.
2. **Co składa się na PWA?** — HTTPS + manifest + SW; co dają, czego brakuje vs natywne.
3. **Opisz cykl życia Service Workera.** — install/waiting/activate; czemu użytkownik widzi starą wersję po deployu.
4. **Jakie strategie cache znasz i kiedy które?** — cache-first/network-first/SWR + offline fallback.
5. **Jak zaimplementujesz offline dla formularza?** — kolejka mutacji w IndexedDB + Background Sync/retry po `online`.

## Dalsza lektura

- [web.dev: Learn PWA](https://web.dev/learn/pwa)
- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Workbox](https://developer.chrome.com/docs/workbox)
- [Jake Archibald: The Service Worker lifecycle](https://web.dev/articles/service-worker-lifecycle)
