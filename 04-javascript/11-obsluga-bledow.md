# Obsługa błędów

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Asynchroniczność](./08-asynchronicznosc.md), [Obiekty i prototypy](./05-obiekty-i-prototypy.md)

Różnica między juniorem a seniorem rzadko leży w happy path — leży w tym, co się dzieje, gdy coś pęka. Ten plik obejmuje mechanikę wyjątków, błędy asynchroniczne i **strategię**: które błędy łapać, gdzie i po co.

## Mechanika: throw, try/catch/finally

```js
try {
  risky();
} catch (err) {
  if (err instanceof ValidationError) { … }   // obsługuj wybiórczo
  else throw err;                              // resztę przepuszczaj (re-throw)
} finally {
  cleanup();                                   // ZAWSZE: sukces, błąd, nawet return
}
```

- Rzucić można wszystko, ale **rzucaj tylko `Error` (lub podklasy)** — string nie ma stacka.
- `catch` bez parametru: `catch { … }` — gdy powód nieistotny.
- `finally` wykonuje się zawsze; `return` w finally nadpisuje wynik/wyjątek (antywzorzec).
- Wyjątek nieobsłużony wędruje w górę call stacka; na szczycie → `window.onerror` i crash bieżącego taska (reszta strony żyje — [event loop](./09-event-loop.md)).

## Typy błędów i błędy własne

Wbudowane: `TypeError` (operacja na złym typie — 90% błędów runtime), `ReferenceError` (niezadeklarowana zmienna/TDZ), `SyntaxError`, `RangeError`, `AggregateError` (z `Promise.any`).

**Własne klasy błędów** — żeby dało się obsługiwać wybiórczo i przenosić kontekst:

```js
class HttpError extends Error {
  constructor(response, message = `HTTP ${response.status}`) {
    super(message, { cause: response });   // cause = łańcuch przyczyn (ES2022)
    this.name = 'HttpError';
    this.status = response.status;
  }
}
```

- `error.cause` — opakowuj błędy niskopoziomowe w domenowe bez gubienia oryginału.
- `error.stack` — stack trace (niestandardowy formalnie, wszędzie dostępny); w produkcji czytelny dzięki [source maps](../14-narzedzia/03-bundlery-i-proces-budowania.md).

## Błędy asynchroniczne

**Promises/async-await** — [asynchroniczność](./08-asynchronicznosc.md) opisuje mechanikę; tu zasady:

```js
try {
  const data = await load();
} catch (err) { … }                    // łapie rejection

load().catch(err => …);                // odpowiednik dla stylu then

// PUŁAPKA: try/catch NIE złapie, jeśli nie ma await
try {
  load();          // fire-and-forget — rejection poleci jako unhandled
} catch { }        // martwy kod dla błędów async
```

- `setTimeout(() => { throw … })` — wyjątku **nie złapie** otaczający try/catch (inny task!); łap wewnątrz callbacka.
- Handlery globalne (telemetria — [monitoring](../16-devops-dla-frontendu/03-monitoring-i-obserwabilnosc.md)):

```js
window.addEventListener('error', e => report(e.error));
window.addEventListener('unhandledrejection', e => report(e.reason));
```

## Strategia — kiedy łapać, a kiedy nie

Zasady senior-level:

1. **Łap tam, gdzie umiesz zareagować** (retry, fallback, komunikat, stan pusty). Catch, który tylko loguje i połyka — ukrywa bugi.
2. **Fail fast dla błędów programisty** (bug w kodzie: TypeError, zła konfiguracja) — niech lecą do globalnego handlera i telemetrii. **Obsługuj błędy operacyjne** (sieć, walidacja, brak uprawnień) — to normalne stany aplikacji.
3. **Warstwa API tłumaczy błędy techniczne na domenowe** (`HttpError` → `SesjaWygasla`), UI decyduje, co pokazać — [wzorce pracy z danymi](../12-komunikacja-z-api/04-wzorce-pracy-z-danymi.md).
4. **Error boundaries** na poziomie widoków (koncepcja frameworkowa): błąd renderu jednego widżetu nie może wywalać całej aplikacji — [model komponentowy](../07-frameworki-koncepcyjnie/02-model-komponentowy.md).
5. Wyjątki są **od sytuacji wyjątkowych** — nie do sterowania logiką („brak wyniku wyszukiwania" to wartość, nie wyjątek). Alternatywa dla oczekiwanych niepowodzeń: wynik typu `{ ok, value | error }` (result object) — czytelny kontrakt bez try/catch na każdym poziomie.

## Pułapki i częste błędy

- Puste `catch {}` — połknięty błąd znika na zawsze; minimum: log + telemetria.
- `catch (e) { throw new Error(e.message) }` — gubi stack i typ; użyj `cause` albo re-throw oryginału.
- try/catch wokół wywołania async bez await.
- Jeden wielki try/catch na całą funkcję zamiast wokół ryzykownego fragmentu — utrudnia diagnozę, łapie za dużo.
- Pokazywanie userowi surowych komunikatów technicznych (i logowanie do konsoli danych wrażliwych).
- `fetch` „nie rzuca na 404" — [fetch w praktyce](../05-dom-i-web-api/03-fetch-i-http-w-praktyce.md); sprawdzaj `response.ok`.

## Pytania rekrutacyjne

1. **Jak działa propagacja wyjątków w JS — sync i async?** — w górę stacka / w dół łańcucha Promise; różnice i pułapka setTimeout.
2. **Po co własne klasy błędów i pole `cause`?** — selektywna obsługa, kontekst domenowy, łańcuch przyczyn.
3. **Gdzie w aplikacji frontendowej obsługujesz błędy?** — warstwowo: API→domenowe, UI-stany, error boundary, global handler+telemetria.
4. **Kiedy NIE łapać wyjątku?** — gdy nie ma sensownej reakcji; bugi mają fail fast do monitoringu.
5. **Wyjątki vs wartości-wyniki (result) — kompromisy?** — kontrola przepływu vs jawność kontraktu; gdzie które.

## Dalsza lektura

- [MDN: Control flow and error handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)
- [javascript.info: Error handling](https://javascript.info/error-handling)
- [MDN: Error.prototype.cause](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/cause)
