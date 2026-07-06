# Jak działa internet

> **Poziom:** 🟢 podstawowy
> **Wymagana wiedza:** brak — zacznij tutaj

Internet to globalna sieć połączonych ze sobą komputerów, które komunikują się za pomocą ustandaryzowanych protokołów. Dla frontend developera zrozumienie tej warstwy jest fundamentem: każdy request, każdy problem z wydajnością ładowania i każda decyzja o hostingu wynika z tego, jak fizycznie i logicznie przepływają dane.

## Model warstwowy (uproszczony)

Komunikacja w sieci jest podzielona na warstwy — każda rozwiązuje inny problem i nie musi wiedzieć, jak działają pozostałe:

| Warstwa | Protokoły | Za co odpowiada |
|---|---|---|
| Aplikacji | HTTP, DNS, WebSocket | Format i znaczenie danych |
| Transportowa | TCP, UDP, QUIC | Dostarczenie danych do właściwego procesu (porty), niezawodność |
| Internetowa | IP | Adresowanie i routing pakietów między sieciami |
| Łącza danych | Ethernet, Wi-Fi | Fizyczne przesyłanie bitów |

Frontend operuje głównie w warstwie aplikacji ([HTTP](./02-protokol-http.md)), ale warstwy niższe wyjaśniają, *dlaczego* rzeczy działają tak, a nie inaczej (np. czemu nawiązanie połączenia kosztuje czas).

## Adresy IP i porty

- **Adres IP** identyfikuje urządzenie w sieci: IPv4 (`142.250.203.142`, pula praktycznie wyczerpana) i IPv6 (`2a00:1450:401b:810::200e`).
- **Port** identyfikuje konkretny proces/usługę na tym urządzeniu: `443` (HTTPS), `80` (HTTP), `3000`/`5173` (typowe porty dev serverów).
- Para `IP:port` jednoznacznie wskazuje odbiorcę danych. `localhost` (`127.0.0.1`) to adres pętli zwrotnej — Twój własny komputer.

## DNS — książka telefoniczna internetu

Ludzie posługują się domenami (`example.com`), maszyny — adresami IP. **DNS (Domain Name System)** tłumaczy jedno na drugie.

Przebieg zapytania (rekurencyjnego):

1. Przeglądarka sprawdza swój cache, potem cache systemu operacyjnego.
2. Zapytanie idzie do **resolvera** (zwykle u dostawcy internetu lub publiczny, np. `1.1.1.1`, `8.8.8.8`).
3. Resolver odpytuje kolejno: **root serwery** → serwery **TLD** (np. `.com`) → **serwer autorytatywny** domeny.
4. Odpowiedź (adres IP) wraca i jest cache'owana zgodnie z **TTL** (time to live) rekordu.

Rodzaje rekordów DNS omawia plik [Domeny, hosting i CDN](./04-domeny-hosting-cdn.md). Z perspektywy wydajności: pierwsze zapytanie DNS do nowej domeny kosztuje zwykle 20–120 ms — dlatego istnieje `<link rel="dns-prefetch">` (zob. [optymalizacja ładowania](../09-wydajnosc/02-optymalizacja-ladowania.md)).

## TCP vs UDP vs QUIC

- **TCP** — połączeniowy i niezawodny: gwarantuje dostarczenie pakietów w kolejności, retransmituje zgubione. Cena: **3-way handshake** (SYN → SYN-ACK → ACK) przed wysłaniem danych, czyli minimum jedna pełna podróż pakietu (RTT) zanim cokolwiek się zacznie.
- **UDP** — bezpołączeniowy: wysyła datagramy bez gwarancji dostarczenia i kolejności. Szybki, używany tam, gdzie zgubiony pakiet boli mniej niż opóźnienie (streaming, gry, DNS).
- **QUIC** — nowoczesny protokół zbudowany **na UDP**, z wbudowanym szyfrowaniem i niezawodnością bez blokowania całego połączenia. To fundament HTTP/3 (szczegóły: [protokół HTTP](./02-protokol-http.md)).

## TLS/HTTPS — szyfrowanie

**HTTPS = HTTP + TLS.** TLS (Transport Layer Security; potocznie wciąż mówi się „SSL") zapewnia:

1. **Poufność** — dane są zaszyfrowane, nikt po drodze ich nie odczyta.
2. **Integralność** — nikt ich niepostrzeżenie nie zmodyfikuje.
3. **Autentyczność** — certyfikat potwierdza, że rozmawiasz z właściwym serwerem. Certyfikaty wystawiają urzędy certyfikacji (CA, np. Let's Encrypt — za darmo), a przeglądarka weryfikuje łańcuch zaufania.

TLS handshake (negocjacja szyfrów, wymiana kluczy) kosztuje dodatkowe RTT. TLS 1.3 skrócił to do 1 RTT (a przy wznowieniu sesji nawet 0-RTT). Bez HTTPS nie działa wiele API przeglądarki (Service Worker, geolokalizacja, kamera) — HTTPS to dziś standard, nie opcja.

## Co się dzieje po wpisaniu URL? (klasyka rozmów rekrutacyjnych)

Pełna sekwencja od Entera do wyrenderowanej strony:

1. **Parsowanie URL** — przeglądarka rozbija adres na schemat, host, ścieżkę (zob. [anatomia URL](./04-domeny-hosting-cdn.md)).
2. **Sprawdzenie cache** — HSTS, cache DNS, cache HTTP ([szczegóły cache'owania](./02-protokol-http.md)).
3. **DNS lookup** — zamiana domeny na IP (jak wyżej).
4. **Połączenie TCP** — 3-way handshake z serwerem (lub QUIC przy HTTP/3).
5. **TLS handshake** — negocjacja szyfrowania (dla HTTPS).
6. **Request HTTP** — `GET / HTTP/2` z nagłówkami (cookies, `Accept`, `User-Agent`…).
7. **Serwer odpowiada** — status, nagłówki, HTML (być może z CDN, przez load balancer, z cache serwera…).
8. **Parsowanie i renderowanie** — przeglądarka buduje DOM, pobiera zasoby, renderuje piksele (cały ten etap: [jak działa przeglądarka](./03-jak-dziala-przegladarka.md)).

Na rozmowie nie musisz recytować wszystkiego — kluczowe jest pokazanie, że rozumiesz kolejność i **koszt każdego etapu** (każdy RTT to realne milisekundy, zwłaszcza na mobile).

## Pułapki i częste błędy

- Mylenie **internetu** (infrastruktura) z **webem** (usługa oparta o HTTP/HTML działająca na tej infrastrukturze).
- Zakładanie, że DNS jest darmowy czasowo — na słabych sieciach lookup potrafi zająć setki milisekund.
- „HTTPS spowalnia" — mit; z TLS 1.3 i HTTP/2+ narzut jest pomijalny, a korzyści (bezpieczeństwo, dostęp do API, SEO) ogromne.
- Zapominanie, że **latencja ≠ przepustowość**: łącze 1 Gb/s z RTT 200 ms nadal odczuwalnie „muli" przy wielu małych requestach.

## Pytania rekrutacyjne

1. **Co się dzieje po wpisaniu adresu w przeglądarce?** — sekwencja wyżej; warto akcentować cache i koszty RTT.
2. **Czym różni się TCP od UDP i gdzie się ich używa?** — niezawodność+kolejność vs szybkość; HTTP(≤2) na TCP, HTTP/3/streaming/DNS na UDP.
3. **Jak działa DNS i co to jest TTL?** — hierarchia resolver → root → TLD → autorytatywny; TTL steruje czasem cache'owania rekordu.
4. **Co daje HTTPS poza szyfrowaniem?** — integralność danych i uwierzytelnienie serwera (certyfikaty, łańcuch zaufania).
5. **Dlaczego pierwsza wizyta na stronie jest wolniejsza niż kolejne?** — zimne cache: DNS, TCP/TLS handshake, brak cache HTTP; kolejne wizyty pomijają większość etapów.

## Dalsza lektura

- [MDN: How the web works](https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Web_standards/How_the_web_works)
- [Cloudflare Learning: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [web.dev: TCP, UDP i QUIC w kontekście wydajności](https://web.dev/articles/performance-http2)
