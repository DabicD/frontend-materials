# Animacje i transformacje

> **Poziom:** 🟡 średni
> **Wymagana wiedza:** [Jak działa przeglądarka](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md) (pipeline renderowania)

Animacje w CSS to dwa mechanizmy (`transition` — między dwoma stanami, `animation` — sekwencje z keyframes) plus `transform` jako najwydajniejszy sposób zmiany wyglądu. Klucz do dobrych animacji: **animuj tylko to, co omija layout i paint**.

## `transform`

```css
.el {
  transform: translateX(20px) scale(1.1) rotate(3deg);
  transform-origin: center bottom;
}
```

- Funkcje: `translate/X/Y/Z`, `scale`, `rotate`, `skew`, `matrix`; 3D: `translate3d`, `rotateY`, `perspective`.
- **Kolejność funkcji ma znaczenie** (translate→rotate ≠ rotate→translate).
- Transform **nie wpływa na layout** — sąsiedzi nie drgną (wizualne przesunięcie, miejsce w flow zostaje).
- Tworzy stacking context i containing block dla `absolute`/`fixed` potomków ([układ dokumentu](./03-uklad-dokumentu.md)) — źródło „czemu mój fixed się zepsuł".
- Można animować indywidualnie: właściwości `translate`, `rotate`, `scale` (bez wspólnego `transform` — łatwiejsze animowanie niezależne).

## `transition` — animacja między stanami

```css
.button {
  background: navy;
  transition: background-color 200ms ease-out, transform 150ms ease;
}
.button:hover { background: royalblue; transform: translateY(-2px); }
```

- Składnia: `property duration timing-function delay`. Nie używaj `transition: all` — wydajność i niespodziewane animacje.
- **Timing functions:** `ease` (domyślna), `ease-out` (naturalna dla wejść/interakcji), `ease-in-out`, `linear` (postęp, obroty), `cubic-bezier(...)`, `steps(n)` (sprite'y, maszynopis), oraz nowa `linear(...)` umożliwiająca spring-like easing.
- Transition wymaga **zmiany wartości animowalnej** — nie zadziała z/do `display: none` ani `height: auto` (patrz niżej).

**Nowości rozwiązujące stare bóle:**
- `@starting-style` — stan początkowy dla elementu, który właśnie pojawił się w DOM (animacje wejścia bez JS).
- `transition-behavior: allow-discrete` — przejścia z udziałem `display`.
- `interpolate-size: allow-keywords` / `calc-size()` — animowanie do `height: auto`.

## `animation` + `@keyframes` — sekwencje

```css
@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50%      { transform: scale(1.05); }
}
.badge {
  animation: pulse 2s ease-in-out infinite;
  /* name duration timing delay iteration-count direction fill-mode play-state */
}
```

- `animation-fill-mode: forwards` — zatrzymaj się na ostatniej klatce (inaczej wraca do stanu bazowego).
- `animation-direction: alternate`, `animation-play-state: paused`.
- Animacje sterowane scrollem: `animation-timeline: scroll()/view()` (scroll-driven animations — progres bary, reveal on scroll bez JS; wsparcie rosnące).
- Z JS: Web Animations API (`el.animate(keyframes, options)`) — te same możliwości imperatywnie, plus kontrola (pause/reverse/playbackRate).

## Wydajność — compositor albo śmierć

Z [pipeline'u renderowania](../01-fundamenty-sieci/03-jak-dziala-przegladarka.md): zmiana geometrii = layout+paint+composite; `transform`/`opacity` = zwykle sam composite (wątek kompozytora, poza main threadem — animacja płynie nawet, gdy JS mieli).

- **Animuj: `transform`, `opacity`** (+ `filter`, `clip-path` — paint, ale bez layoutu).
- **Nie animuj: `width`, `height`, `top`, `left`, `margin`, `font-size`** — layout w każdej klatce.
- Zamienniki: `left: 100px` → `translateX(100px)`; „rozwijanie" wysokości → `grid-template-rows: 0fr → 1fr` (z `min-height: 0` na dziecku) albo `interpolate-size`.
- `will-change: transform` — hint tworzący warstwę kompozytora z wyprzedzeniem. Używaj chirurgicznie i sprzątaj; nadużycie = eksplozja pamięci GPU.

## Dostępność i UX ruchu

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

- Szanuj `prefers-reduced-motion` (choroba lokomocyjna, przedsionkowa) — [podstawy a11y](../10-dostepnosc/01-podstawy-a11y.md).
- Dobre czasy interfejsu: 100–300 ms; wejścia `ease-out`, wyjścia `ease-in`; ruch ma **komunikować** (skąd-dokąd), nie ozdabiać.

## Pułapki i częste błędy

- `transition: all 0.3s` — animuje rzeczy przypadkowe (w tym layoutowe).
- Animacja `height: 0 → auto` „nie działa" — auto nie interpoluje; triki: grid rows 0fr→1fr, `max-height` (kiepskie), `interpolate-size`.
- Brak `animation-fill-mode: forwards` — element „skacze" po zakończeniu.
- Animowanie `box-shadow` wprost (drogi paint) — animuj `opacity` pseudoelementu z cieniem.
- Transform na rodzicu psuje `position: fixed` dziecka.
- Zapominanie o `prefers-reduced-motion`.

## Pytania rekrutacyjne

1. **Dlaczego `transform`/`opacity` są wydajne?** — omijają layout/paint, działają na wątku kompozytora (GPU).
2. **`transition` vs `animation` — kiedy które?** — reakcja na zmianę stanu (A→B) vs samodzielna sekwencja/zapętlenie.
3. **Jak animować rozwijanie do naturalnej wysokości?** — grid 0fr→1fr / `interpolate-size: allow-keywords`; czemu `height: auto` nie interpoluje.
4. **Co robi `will-change` i jakie ma koszty?** — promocja do warstwy; pamięć GPU, używać punktowo.
5. **Jak wspierasz użytkowników wrażliwych na ruch?** — `prefers-reduced-motion` + projektowanie ruchu funkcjonalnego.

## Dalsza lektura

- [MDN: Using CSS transitions / animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transitions/Using_CSS_transitions)
- [web.dev: Animations guide](https://web.dev/articles/animations-guide)
- [Josh Comeau: An Interactive Guide to CSS Transitions](https://www.joshwcomeau.com/animation/css-transitions/)
