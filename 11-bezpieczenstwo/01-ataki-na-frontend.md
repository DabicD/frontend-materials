# Ataki na frontend

> **Poziom:** 🔴 zaawansowany (wiedza obowiązkowa)
> **Wymagana wiedza:** [DOM — manipulacja](../05-dom-i-web-api/01-dom-manipulacja.md), [Przechowywanie danych](../05-dom-i-web-api/04-przechowywanie-danych.md)

Frontend jest kodem uruchamianym na **wrogim urządzeniu**, w przeglądarce, którą kontroluje potencjalny atakujący. Zrozumienie ataków to warunek pisania bezpiecznego kodu — obrona bez znajomości zagrożeń jest ślepa. Ten plik opisuje ataki; obrony zbiera [mechanizmy obronne](./02-mechanizmy-obronne.md).

## Złota zasada

**Nigdy nie ufaj danym z zewnątrz** (użytkownik, URL, API, `postMessage`, storage). Frontend nie jest granicą bezpieczeństwa — **walidacja i autoryzacja są na serwerze** ([REST](../12-komunikacja-z-api/01-rest.md), [uwierzytelnianie](./03-uwierzytelnianie-i-autoryzacja.md)); frontend chroni głównie **użytkownika w jego przeglądarce**.

## XSS (Cross-Site Scripting) — zagrożenie nr 1

Wstrzyknięcie i wykonanie **cudzego JavaScriptu w kontekście Twojej strony**. Taki skrypt ma pełnię praw użytkownika: czyta [localStorage](../05-dom-i-web-api/04-przechowywanie-danych.md), cookies bez `HttpOnly`, DOM, wysyła requesty jako zalogowany user, podmienia UI (fałszywy formularz logowania).

**Rodzaje:**
- **Stored** — złośliwy payload zapisany na serwerze (komentarz, nazwa), serwowany innym. Najgroźniejszy (zasięg).
- **Reflected** — payload w URL/parametrach odbity w odpowiedzi (link w phishingu).
- **DOM-based** — czysto kliencki: JS bierze dane z `location`/inputu i wstawia do DOM niebezpiecznie.

**Wektory (gdzie powstaje):**

```js
el.innerHTML = userInput;                    // ✘ parsuje HTML/skrypty
el.insertAdjacentHTML('beforeend', data);    // ✘
element.setAttribute('href', userInput);     // ✘ 'javascript:...'
eval(data); new Function(data);              // ✘ nigdy z danymi
a.href = userUrl;                            // ✘ jeśli userUrl = "javascript:alert(1)"
// frameworki: dangerouslySetInnerHTML / v-html — furtki omijające auto-escaping
```

**Obrona (skrót — pełna w [mechanizmach](./02-mechanizmy-obronne.md)):** `textContent` zamiast `innerHTML`; sanityzacja HTML (DOMPurify) gdy HTML konieczny; walidacja URL (tylko `http(s):`); auto-escaping frameworka + [CSP](./02-mechanizmy-obronne.md) jako druga linia.

## CSRF (Cross-Site Request Forgery)

Atakujący zmusza przeglądarkę **zalogowanej** ofiary do wysłania requestu do Twojego serwisu — wykorzystując, że cookies [dołączają się automatycznie](../05-dom-i-web-api/04-przechowywanie-danych.md). Ofiara wchodzi na złą stronę, a ta cicho wysyła `POST /transfer` na Twój bank; przeglądarka dokleja cookie sesji.

- Dotyczy uwierzytelniania **cookie-based**; tokeny w nagłówku `Authorization` (dodawane ręcznie przez JS) są z natury odporne — ale wtedy dochodzi ryzyko XSS. Kompromis: [uwierzytelnianie](./03-uwierzytelnianie-i-autoryzacja.md).
- Obrona: **`SameSite` cookies** (dziś domyślnie `Lax` — duża część problemu znika), tokeny anty-CSRF, sprawdzanie `Origin`/`Referer` ([mechanizmy](./02-mechanizmy-obronne.md)).

## Clickjacking

Twoja strona osadzona w niewidocznym `<iframe>` na stronie atakującego; ofiara „klika przycisk", nie wiedząc, że klika Twój (np. „usuń konto"). Obrona: `X-Frame-Options: DENY` / CSP `frame-ancestors` — [mechanizmy](./02-mechanizmy-obronne.md).

## Prototype pollution

JS-specyficzny: wstrzyknięcie do `Object.prototype` przez klucze `__proto__`/`constructor` w danych (naiwny deep-merge, `JSON` z kontrolowanym kształtem → parsowanie do obiektu). Zanieczyszczenie prototypu wpływa na **wszystkie** obiekty ([prototypy](../04-javascript/05-obiekty-i-prototypy.md)) — może eskalować do XSS/omijania logiki.

```js
merge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'));   // ({}).isAdmin === true (!)
```

Obrona: `Object.create(null)`/`Map` na dane zewnętrzne, blokowanie kluczy `__proto__`/`constructor`/`prototype`, `Object.freeze(Object.prototype)`, sprawdzone biblioteki.

## Supply chain (łańcuch dostaw)

Największe realne ryzyko dziś: **cudzy kod w Twoim bundle'u**. Złośliwa/przejęta paczka npm, zależność zależności, skompromitowany skrypt third-party (analityka, widget) — wykonują się z pełnymi prawami Twojej strony. Głośne incydenty: typosquatting, przejęte popularne paczki, malware w post-install. Obrona: audyt i higiena zależności ([pakiety](../14-narzedzia/02-pakiety-i-zaleznosci.md)), SRI dla skryptów z CDN, CSP ograniczające źródła, minimalizm zależności — [mechanizmy](./02-mechanizmy-obronne.md).

## Pozostałe wektory (świadomość)

- **Open redirect** — `?returnTo=` bez walidacji → przekierowanie na phishing (waliduj cele redirectów!).
- **Wyciek danych do third-party** — dane wrażliwe w URL/logach/analityce; `Referer` wyciekający tokeny z URL.
- **`window.opener`** — `target="_blank"` bez `rel="noopener"` pozwala nowej karcie sterować oknem źródłowym (dziś domyślnie noopener, ale pilnuj).
- **Dane wrażliwe w kodzie klienta** — klucze API/sekrety w bundle'u są **jawne** (DevTools → Sources); nic w froncie nie jest tajne.
- **Regex DoS (ReDoS)**, **postMessage bez sprawdzenia origin**.

## Pułapki i częste błędy

- „Walidujemy na froncie, więc jest bezpiecznie" — request można wysłać z pominięciem UI.
- Sekret/klucz w kodzie klienta „bo schowany w zmiennej env" — build i tak go zawiera.
- `innerHTML` z „zaufanymi" danymi, które okazują się pochodzić od użytkownika (nazwa, bio).
- Wyłączanie escapingu frameworka dla wygody (`v-html`/`dangerouslySetInnerHTML`) bez sanityzacji.
- Instalowanie paczek bez oglądania (liczba pobrań ≠ bezpieczeństwo).

## Pytania rekrutacyjne

1. **Czym jest XSS, jakie ma rodzaje i jak się bronisz?** — wykonanie cudzego JS; stored/reflected/DOM; escaping/sanityzacja/CSP.
2. **Jak działa CSRF i czemu tokeny w nagłówku są odporne?** — auto-dołączanie cookies; nagłówek dodawany ręcznie przez JS.
3. **Dlaczego klucz API w kodzie frontendu to problem?** — bundle jest jawny; nic w kliencie nie jest sekretem.
4. **Co to prototype pollution?** — wstrzyknięcie do Object.prototype; naiwny merge; skutki globalne.
5. **Największe ryzyko bezpieczeństwa nowoczesnego frontendu?** — supply chain (zależności/third-party); jak ograniczasz.

## Dalsza lektura

- [OWASP Top 10](https://owasp.org/www-project-top-ten/) + [OWASP Cheat Sheets (XSS, CSRF)](https://cheatsheetseries.owasp.org/)
- [MDN: Website security](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Server-side/First_steps/Website_security)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) — darmowe laby (XSS, CSRF)
