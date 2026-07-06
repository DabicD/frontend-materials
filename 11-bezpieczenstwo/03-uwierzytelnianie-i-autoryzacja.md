# Uwierzytelnianie i autoryzacja

> **Poziom:** 🔴 zaawansowany
> **Wymagana wiedza:** [Ataki na frontend](./01-ataki-na-frontend.md), [Mechanizmy obronne](./02-mechanizmy-obronne.md), [Przechowywanie danych](../05-dom-i-web-api/04-przechowywanie-danych.md)

**Uwierzytelnianie (authentication)** = „kim jesteś" (logowanie). **Autoryzacja (authorization)** = „co ci wolno" (uprawnienia). Frontend uczestniczy w obu, ale — to sedno — **żadnego nie egzekwuje**: prawdziwa kontrola jest na serwerze. Rola frontendu to bezpieczne przenoszenie poświadczeń i dopasowanie UI.

## Sesje (cookie) vs tokeny (JWT)

**Sesja po stronie serwera:** serwer trzyma stan sesji, klient dostaje **cookie** z identyfikatorem. Cookie [dołącza się automatycznie](../05-dom-i-web-api/04-przechowywanie-danych.md).

- ✚ serwer w pełni kontroluje (natychmiastowe unieważnienie/wylogowanie), identyfikator nie niesie danych.
- ✖ stan po stronie serwera (skalowanie/współdzielenie), podatność na [CSRF](./01-ataki-na-frontend.md) (auto-cookie).

**JWT (JSON Web Token):** podpisany token niosący claims (id, role, exp), weryfikowany podpisem bez sięgania do bazy.

- Budowa: `header.payload.signature` (base64url). **`payload` jest tylko zakodowany, nie zaszyfrowany** — czytelny dla każdego; nigdy nie wkładaj tam sekretów.
- ✚ bezstanowy (skalowanie, mikroserwisy), samo-zawierający.
- ✖ **trudny do unieważnienia** przed wygaśnięciem (stąd krótki `exp` + refresh token + ewentualna denylist); rośnie z claimami.

## Gdzie trzymać token — kluczowa decyzja frontendu

To ulubione pytanie rekrutacyjne, bo nie ma darmowego lunchu — wybór to kompromis XSS vs CSRF:

| Miejsce | Ryzyko | Uwagi |
|---|---|---|
| **`localStorage`** | **XSS** czyta go trywialnie ([ataki](./01-ataki-na-frontend.md)) | wygodne, ale każdy wstrzyknięty skrypt kradnie token |
| **Cookie `HttpOnly; Secure; SameSite`** | **CSRF** (mitygowany SameSite + token anty-CSRF) | JS **nie ma** dostępu — odporne na kradzież przez XSS; rekomendowane |
| Pamięć JS (zmienna) | znika po odświeżeniu | dobre dla access tokena (krótki), refresh w cookie HttpOnly |

**Konsensus branżowy:** wrażliwe poświadczenia w **cookie `HttpOnly`** (JS ich nie odczyta) + obrona CSRF (`SameSite=Lax/Strict`, token anty-CSRF). Popularny wzorzec: krótki **access token w pamięci** + **refresh token w cookie HttpOnly**; odświeżanie w tle. Zauważ: przy silnym XSS atakujący i tak działa jako użytkownik (wysyła requesty z auto-cookie) — dlatego **eliminacja XSS jest ważniejsza** niż spór o miejsce tokena.

## OAuth 2.0 i OpenID Connect (OIDC)

- **OAuth 2.0** — **autoryzacja delegowana**: „pozwól aplikacji X zrobić Y w Twoim imieniu" bez podawania jej hasła (logowanie przez Google, dostęp do API).
- **OIDC** — warstwa **uwierzytelniania** nad OAuth: dodaje **ID token** (kto to jest) — to on realizuje „Zaloguj przez Google".
- **Authorization Code + PKCE** — jedyny właściwy flow dla aplikacji frontendowych (SPA/mobile). PKCE chroni publicznego klienta (bez sekretu) przed przechwyceniem kodu. **Implicit flow jest przestarzały** — nie używaj.
- Rola frontendu: redirect do authorization server, odbiór kodu, wymiana na token (przez PKCE), przechowanie zgodnie z zasadami wyżej. Ciężar bezpieczeństwa niosą biblioteki (oidc-client-ts, Auth.js, SDK dostawców) — nie implementuj OAuth ręcznie.

## Autoryzacja we frontendzie = tylko UX

**Reguła absolutna:** ukrycie przycisku „Usuń" ≠ ochrona. Atakujący pominie UI i uderzy w API bezpośrednio ([fetch](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md)). **Każdy endpoint egzekwuje uprawnienia po stronie serwera.**

Frontend robi warstwę UX na tych samych danych:
- Chowanie/wyłączanie akcji bez uprawnień (czytelniejszy interfejs).
- **Route guards** — redirect niezalogowanych ([routing](../07-frameworki-koncepcyjnie/05-routing-po-stronie-klienta.md)); to wygoda, nie bariera.
- Model uprawnień (role/RBAC, uprawnienia/PBAC) współdzielony z backendem, ale **weryfikowany** przez backend.
- Obsługa `401` (przeloguj/odśwież token) i `403` (brak uprawnień — komunikat) w [warstwie API](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md).

## Flagi cookie — ściąga

```http
Set-Cookie: session=…; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

- **`HttpOnly`** — niedostępne dla JS (anty-XSS-kradzież). **`Secure`** — tylko HTTPS. **`SameSite`** — `Lax` (domyślne; blokuje cross-site POST — anty-CSRF), `Strict` (max ochrona, gorszy UX linków), `None` (wymaga `Secure`; do scenariuszy cross-site).
- Prefiksy `__Host-`/`__Secure-` wymuszają bezpieczną konfigurację.

## Pułapki i częste błędy

- Autoryzacja tylko w UI („schowaliśmy przycisk") bez kontroli na endpoincie — krytyczna dziura.
- Wrażliwe dane/sekrety w payloadzie JWT (jest czytelny) lub tokeny w URL (wyciek w `Referer`/logach/historii).
- Token w localStorage przy realnym ryzyku XSS.
- Ręczna implementacja OAuth/JWT zamiast sprawdzonych bibliotek.
- Brak obsługi wygaśnięcia/odświeżania — użytkownik „wylogowywany" losowo albo sesja żyje wiecznie.
- Zaufanie do danych z tokena bez weryfikacji podpisu po stronie serwera.

## Pytania rekrutacyjne

1. **Uwierzytelnianie vs autoryzacja — różnica i rola frontendu?** — kim/co wolno; front przenosi i dopasowuje UI, nie egzekwuje.
2. **Gdzie przechowasz token i jakie są kompromisy?** — localStorage (XSS) vs cookie HttpOnly (CSRF); rekomendacja + dlaczego XSS jest sednem.
3. **Czym jest JWT i co znaczy, że payload nie jest szyfrowany?** — podpisany, czytelny; brak sekretów, unieważnianie.
4. **Który flow OAuth dla SPA i czemu nie implicit?** — Authorization Code + PKCE; bezpieczeństwo publicznego klienta.
5. **Czemu autoryzacja we froncie to tylko UX?** — API osiągalne z pominięciem UI; egzekwowanie na serwerze.

## Dalsza lektura

- [OWASP: Authentication / Session Management / JWT Cheat Sheets](https://cheatsheetseries.owasp.org/)
- [OAuth 2.0 for Browser-Based Apps (IETF BCP)](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps)
- [web.dev: SameSite cookies explained](https://web.dev/articles/samesite-cookies-explained)
