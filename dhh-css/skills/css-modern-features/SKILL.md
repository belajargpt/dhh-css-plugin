---
name: css-modern-features
description: "Use when you want modern native CSS to replace JavaScript or a preprocessor — cascade layers (@layer), native nesting with &, :has() parent/quantity selector, :is()/:where(), @starting-style and transition allow-discrete enter/exit on <dialog> and ::backdrop, container queries (container-type, cqi, @container), clamp()/min()/max() and dvw/dvh fluid sizing, logical properties (padding-inline, margin-block, inset-inline-start), color-mix() and light-dark() theming, CSS masks for icons and the no-JS loading spinner, mix-blend-mode highlighter/circled-text, capability queries (any-hover, pointer: fine/coarse), and character-based breakpoints (max-width: 100ch). Framework-agnostic 37signals-style vanilla CSS — zero build step, equally usable in plain static HTML or any stack."
---

# Modern CSS Features

> The browser is the framework now. Reach for a feature before a dependency — most of what we used JavaScript and Sass for ships natively today.

37signals built three products — Campfire, Writebook, Fizzy — on ~14,000 lines of CSS, **zero build tools**: no Sass, no PostCSS, no Tailwind, no compile step. This skill is the catalog of native features that make that possible. Every example below is real, lifted from those stylesheets. Load the CSS however your stack loads CSS — a bundler entry that imports each file, `<link>` tags, or your framework's global stylesheet import. Two conventions run throughout: **cascade layers** so source order never decides the winner, and **logical properties** (`inline-size`, `padding-inline`, `margin-block`, `inset-*`) so layouts mirror under right-to-left writing modes for free.

## Cascade layers (`@layer`)

Declare layer order once; later layers always beat earlier ones regardless of selector specificity. Specificity wars and `!important` evaporate — a utility in the `utilities` layer wins over a component in the `components` layer even if the component used an ID.

```css
@layer reset, base, components, modules, utilities;

@layer components {
  .btn { /* always lower priority than utilities */ }
}

@layer utilities {
  .hide { display: none; } /* always wins over components, no !important */
}
```

**Payoff:** You decide priority by *layer*, not by counting selector weight. New components can't accidentally out-specify a utility. **Support:** all evergreen browsers (Chrome/Edge 99+, Firefox 97+, Safari 15.4+).

## Native nesting with `&`

Group a component's states, descendants, pseudo-elements, and media queries in one block — no preprocessor. Place `&` *after* an ancestor selector to mean "this element, when inside that ancestor."

```css
a {
  text-decoration: none;

  &:not([class]) {
    color: var(--color-link);
    text-decoration: underline;
    text-decoration-skip-ink: auto;
  }
}

/* Context-aware theming: "this element, in dark mode" */
::selection {
  background: var(--color-selected);

  html[data-theme="dark"] & {
    background-color: var(--color-selected-dark);
  }
}
```

**Payoff:** Co-locate everything about an element — Sass nesting without Sass. The dark-theme rule reads as plain nesting against the `[data-theme="dark"]` contract; whatever sets that attribute is irrelevant to the CSS. **Support:** Chrome/Edge 112+, Firefox 117+, Safari 17.2+.

## `:is()` and `:where()`

`:is(a, b, c)` groups selectors into one rule and takes the specificity of its **most specific** argument. `:where(...)` is identical but always contributes **zero** specificity — the override valve.

```css
/* Shared transitions/focus for a whole family, in one rule */
:is(a, button, input, textarea, .switch, .btn) {
  transition: 100ms ease-out;
  transition-property: background-color, border-color, box-shadow, filter, outline;
  touch-action: manipulation;

  &:where(:focus-visible) {
    border-radius: 0.25ch;
    outline: var(--focus-ring-size) solid var(--focus-ring-color);
    outline-offset: var(--focus-ring-offset);
  }
}

/* :where() drops an ID's 1-0-0 specificity to 0-0-0 so utilities still win */
:where(#main) {
  inline-size: 100dvw;
  margin-inline: auto;
  max-inline-size: 100dvw;
  text-align: center;
}
```

**Payoff:** `:is()` kills repetition; `:where()` is how you write structural defaults (even on IDs) that any later class overrides without `!important`. **Support:** all evergreen browsers (Chrome/Edge 88+, Firefox 78+, Safari 14+).

## `:has()` — the parent / quantity selector

Style an element based on what it *contains* or what a *sibling* does. This is the feature that replaces JavaScript bookkeeping and conditional templating logic with a declarative selector.

```css
/* State-driven layout: the kanban grid re-templates when a column is expanded */
.card-columns {
  container-type: inline-size;
  display: grid;
  grid-template-columns: 1fr var(--column-width-expanded) 1fr;

  &:has(.card-columns__left .cards:not(.is-collapsed), .card-columns__right .cards:not(.is-collapsed)) {
    grid-template-columns: auto var(--column-width-expanded) auto;
  }

  &:has(.cards) {
    min-block-size: 20lh;
  }
}

/* Lock body scroll while a dialog deep inside is open — no scroll-lock library */
html:has(.lightbox[open]) {
  overflow: clip;
}

/* Collapse-empty: hide a container when ALL its children are hidden */
.popup:has(.popup__item[hidden]):not(:has(.popup__item:not([hidden]))) {
  display: none;
}
```

**Quantity / counting** — combine `:has()` with `:nth-child()` to react to *how many* children exist. The avatar group lays out differently for one, two, or three avatars (full verbatim version in the reference):

```css
.avatar__group {
  &:where(:has(> :last-child:nth-child(2))) { /* exactly two avatars */
    --avatar-size: 3.5ch;
  }
  &:where(:has(> :last-child:nth-child(3))) { /* exactly three avatars */
    --avatar-size: 2.5ch;
  }
}
```

**Payoff:** The parent reacts to its descendants with no wrapper class, no ancestor-class toggling, no count passed in from a template. "Is anything expanded?", "are there unread items?", "how many children?" become pure CSS. **Support:** Chrome/Edge 105+, Firefox 121+, Safari 15.4+.

## `@starting-style` + `transition ... allow-discrete`

CSS-only enter **and** exit animations for top-layer elements (`<dialog>`, popovers, lightbox) with no JS class toggling. `allow-discrete` lets the discrete `display` and `overlay` properties participate so the element stays visible through its exit; `@starting-style` supplies the "first frame" so the open animation plays the very first time the element appears. Works on `::backdrop` too.

```css
@layer components {
  :is(.dialog) {
    border: 0;
    opacity: 0;
    transform: scale(0.2);
    transform-origin: top center;
    transition: var(--dialog-duration) allow-discrete;
    transition-property: display, opacity, overlay, transform;

    &::backdrop {
      background-color: var(--color-black);
      opacity: 0;
      transition: var(--dialog-duration) allow-discrete;
      transition-property: display, opacity, overlay;
    }

    &[open] {
      opacity: 1;
      transform: scale(1);

      &::backdrop { opacity: 0.5; }
    }

    @starting-style {
      &[open] { opacity: 0; transform: scale(0.2); }
      &[open]::backdrop { opacity: 0; }
    }
  }
}
```

**Payoff:** A dialog animates in and out purely from the presence of the `[open]` attribute. No `requestAnimationFrame`, no `transitionend` listener, no enter/leave classes — the only contract is "the element gets `[open]` when shown." **Support:** Chrome/Edge 117+, Safari 17.4+, Firefox 129+.

## Container queries

Make an element a query container with `container-type: inline-size`, then size its contents in container units (`cqi` = 1% of container inline size) or query its width with `@container`. A card responds to the **column it lives in**, not the viewport.

```css
.card-columns {
  --cards-gap: min(1.2cqi, 1.7rem);
  container-type: inline-size;
  display: grid;
}

.cards .card {
  /* font tracks the column width, with a hard floor */
  font-size: clamp(0.6rem, 0.85cqi, 100px);
}
```

**Payoff:** True component-level responsiveness. The same card behaves identically in a wide desktop board or a narrow mobile column — no viewport breakpoints to juggle, because it measures its own container. **Support:** Chrome/Edge 105+, Firefox 110+, Safari 16+.

## `clamp()` / `min()` / `max()` and viewport units

`clamp(min, preferred, max)` is a fluid value with a hard floor and ceiling — three breakpoints collapsed into one continuously-fluid declaration. `min()`/`max()` express "smaller/larger of two constraints" inline. `dvw`/`dvh` are the dynamic viewport units that account for mobile browser chrome.

```css
/* Fluid type, bounded — no breakpoint cliffs */
font-size: clamp(var(--text-xx-small), 3.15cqi, var(--text-small));

/* "the smaller of: ideal width, or viewport minus gutters" */
max-inline-size: min(50ch, calc(100vw - (var(--inline-space) * 2)));

/* dynamic viewport width — correct under mobile address-bar collapse */
inline-size: 100dvw;
max-inline-size: 100dvw;
```

**Payoff:** Smooth scaling instead of jumpy `@media` steps. One declaration replaces a stack of breakpoints and never breaks its bounds; `dvw`/`dvh` fix the "100vh is too tall on mobile" bug. **Support:** `clamp`/`min`/`max` all evergreen; `dvw`/`dvh` Chrome/Edge 108+, Firefox 101+, Safari 15.4+.

## Logical properties

Use `inline-size`/`block-size`, `padding-inline`/`margin-block`, and `inset-inline-start`/`inset-block-end` instead of physical `width`/`height`/`left`/`top`. The layout mirrors automatically under RTL — no separate stylesheet.

```css
:root {
  --inline-space: 1ch;   /* horizontal: one character width */
  --block-space: 1rem;   /* vertical: one root em */
}

.component {
  padding-inline: var(--inline-space);
  margin-block: var(--block-space);
}

.position-sticky {
  position: sticky;
  inset: var(--inset, 0 auto auto auto);
}
```

| Physical             | Logical                                    |
| -------------------- | ------------------------------------------ |
| `width` / `height`   | `inline-size` / `block-size`               |
| `padding-left/right` | `padding-inline-start` / `-end`            |
| `margin-top/bottom`  | `margin-block-start` / `-end`              |
| `left` / `top`       | `inset-inline-start` / `inset-block-start` |

**Payoff:** Direction-agnostic by default — switch the document to RTL and start/end swap automatically. Spacing tokens read as intent (`inline` vs `block`), not a hard-coded side. **Support:** all evergreen browsers (Chrome/Edge 89+, Firefox 66+, Safari 15+).

## `color-mix()` and `light-dark()`

`color-mix()` derives colors at runtime — tints, shades, alpha — from a single base token, so a whole component palette flows from one variable. `light-dark()` returns one of two values depending on the resolved `color-scheme`, collapsing a `@media (prefers-color-scheme)` block into a single function call.

```css
/* Derive an entire card palette from one --card-color */
.card {
  --card-color: oklch(var(--lch-blue-dark));
  --card-bg:     color-mix(in srgb, var(--card-color) 4%, var(--color-canvas));
  --card-text:   color-mix(in srgb, var(--card-color) 30%, var(--color-ink));
  --card-border: color-mix(in srgb, var(--card-color) 33%, transparent);
}

/* One token, both themes — no media query */
:root { color-scheme: light dark; }
.surface {
  background: light-dark(oklch(98% 0 0), oklch(22% 0 0));
  color:      light-dark(oklch(22% 0 0), oklch(92% 0 0));
}
```

**Payoff:** Change one variable, the whole derived palette moves with it. No hand-maintained tint/shade ramps, no duplicated dark-mode color tables. **Support:** `color-mix()` Chrome/Edge 111+, Firefox 113+, Safari 16.2+; `light-dark()` Chrome/Edge 123+, Firefox 120+, Safari 17.5+.

## CSS masks — icons and the no-JS spinner

A mask punches the visible shape out of a solid `currentColor` fill. Icons become one base class plus a `--svg` line, recoloring for free. The loading spinner is drawn entirely with three stacked radial gradients animated by sliding the mask — no SVG file, no GIF, no JavaScript.

```css
@layer components {
  .icon {
    background-color: currentColor;   /* color comes from text/link/theme */
    block-size: var(--icon-size, 1em);
    inline-size: var(--icon-size, 1em);
    display: inline-block;
    mask-image: var(--svg);
    mask-position: center;
    mask-repeat: no-repeat;
    mask-size: var(--icon-size, 1em);
  }
  .icon--bell  { --svg: url("bell.svg"); }
  .icon--check { --svg: url("check.svg"); }
}

/* Three-dot spinner: one ::before, masked currentColor, animated mask-position */
@layer components {
  .spinner::before {
    --mask: no-repeat radial-gradient(#000 68%, #0000 71%);
    --dot-size: 1.25em;
    -webkit-mask: var(--mask), var(--mask), var(--mask);
    -webkit-mask-size: 28% 45%;
    animation: submitting 1.3s infinite linear;
    aspect-ratio: 8/5;
    background: currentColor;
    content: "";
    inline-size: var(--dot-size);
  }
}
@keyframes submitting {
  0%   { -webkit-mask-position: 0% 0%,   50% 0%,   100% 0% }
  50%  { -webkit-mask-position: 0% 100%, 50% 100%, 100% 100% }
  100% { -webkit-mask-position: 0% 0%,   50% 0%,   100% 0% }
}
```

**Payoff:** An icon inside a red button is red; in dark mode it's the inverted ink — no per-theme icon variants, no editing SVG `fill`. The spinner costs one element, scales from a single `--dot-size`, and animates only `mask-position` (compositor-cheap). Full verbatim keyframes are in the reference. **Support:** masks ship cross-browser but Safari/WebKit still require the `-webkit-mask*` prefix — declare both.

## `mix-blend-mode`

Blend a layer into whatever sits behind it — highlighter sweeps, circled-text marks, glows. The key move is swapping blend mode per theme: `multiply` darkens over a light canvas, `screen` lightens over a dark one, so "ink" reads correctly in both.

```css
@layer components {
  .circled-text span {
    opacity: 0.5;
    mix-blend-mode: multiply;     /* darken on light */

    html[data-theme="dark"] & {
      mix-blend-mode: screen;     /* lighten on dark */
    }
    @media (prefers-color-scheme: dark) {
      html:not([data-theme]) & { mix-blend-mode: screen; }
    }
  }
}
```

**Payoff:** The effect interacts with any background instead of needing a per-color variant. One blend-mode swap keeps the darken-on-light / lighten-on-dark behavior correct across themes — no recoloring. Full circled-text drawing is in the reference. **Support:** all evergreen browsers (Chrome/Edge 41+, Firefox 32+, Safari 8+).

## Capability queries

Branch on what the *input device* can do, not on screen width. `any-hover: hover` gates hover affordances so touch users never get stuck-on hover states; `pointer: fine` / `coarse` distinguishes a precise pointer from a finger.

```css
/* Reveal a control only on devices that can hover */
@media (any-hover: hover) and (pointer: fine) {
  .row .controls { opacity: 0; }
  .row:hover .controls { opacity: 1; }
}

/* Always show it where there is no hover (touch) */
@media (any-hover: none) {
  .show-on-touch { display: unset; }
  .hide-on-touch { display: none; }
}
```

**Payoff:** A tooltip a touch user can't trigger never appears; a keyboard hint they can't use is hidden. UI branches on capability instead of guessing from viewport width. (Prefer `any-hover` over `any-pointer` — the latter is misreported on many devices.) **Support:** all evergreen browsers (Chrome/Edge 41+, Firefox 64+, Safari 9+).

## Character-based breakpoints

Break on `ch` (character width), not device pixels: `@media (max-width: 100ch)`. The layout responds to the **type**, so it does the right thing on any device — and also when the user simply enlarges the font past a threshold.

```css
/* Wide enough for a sidebar, measured in characters of text */
@media (min-width: 100ch) {
  .layout { grid-template-columns: 1fr 20ch; }
}

@media (max-width: 100ch) {
  .layout { grid-template-columns: 1fr; }
}
```

**Payoff:** "Type is the heart of web pages, so the layout responds to it." A `ch`-relative breakpoint adapts to font size and zoom, not just to a device class — far more robust than `px` device-width breakpoints that lie the moment someone bumps their base font. **Support:** `ch` units and `@media` work everywhere; this is a technique, not a new feature.

## The frontier — coming next

Three more native features are stabilizing as the next layer of "CSS replaces JS." Use them with progressive enhancement (e.g. behind `@supports`) until support is universal.

| Feature | What it replaces | Status (mid-2026) |
| --- | --- | --- |
| **View Transitions** (`view-transition-name`, `::view-transition`) | JS-driven page/state crossfades and shared-element animations | Same-document: broad evergreen support. Cross-document: rolling out. |
| **Scroll-driven animations** (`animation-timeline: scroll()` / `view()`) | scroll-listener JS for progress bars, reveal-on-scroll, parallax | Chrome/Edge shipped; Safari/Firefox in progress. |
| **Anchor positioning** (`anchor()`, `position-anchor`, `@position-try`) | JS popover/tooltip positioning libraries | Chrome/Edge shipped; others in progress. |

## When to use

- **Reach for these before adding a dependency.** A `:has()` selector, a `@starting-style` transition, or a CSS mask deletes the JavaScript or preprocessor you were about to reach for.
- **Layer everything.** Wrap component rules in `@layer components` and utilities in `@layer utilities` so the cascade is decided by layer order, not specificity accidents.
- **Default to logical properties and `ch`/`cqi`** so layouts are direction- and type-relative, not pixel-and-side hard-coded.

## Anti-Patterns

- **`!important` to win a specificity fight** → use `@layer` ordering or `:where()` (0 specificity).
- **JS class toggling for enter/exit** → `@starting-style` + `allow-discrete` against an `[open]`/attribute contract.
- **A scroll-lock library** → `html:has(.dialog[open]) { overflow: clip; }`.
- **Counting children in a template/controller to pick a layout** → `:has(> :last-child:nth-child(n))`.
- **Device-width (`px`) breakpoints for component layout** → container queries (`cqi`) for components, `ch` breakpoints for text-driven layout.
- **Shipping an SVG/GIF spinner or per-theme icon set** → mask `currentColor`; it recolors itself.
- **Parallel dark-mode color tables** → derive with `color-mix()` / `light-dark()` from one token.

## Related Skills

- **css-design-system** — the overall no-build architecture, file organization, and `@layer` strategy these features plug into.
- **css-design-tokens** — the OKLCH custom-property and spacing-token system (`--lch-*`, `--inline-space`) referenced throughout.
- **css-components** — semantic, variable-driven components (`.btn`, `.card`) that consume these features.
- **css-guardrails** — the rules and review checklist that keep usage of these features consistent and `!important`-free.
