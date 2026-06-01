# Foundation starter files (full, ready-to-paste)

These are the complete foundation files for a blank-slate 37signals-style design
system. Every value is reproduced verbatim from the mined Campfire / Writebook /
Fizzy stylesheets — real OKLCH triplets, real property names, no invented
approximations. They are **pure native CSS**: no build step, no Sass, no PostCSS,
no Tailwind. They read identically in a single static `.html` file or any stack.

Create four flat files. Load the tokens file first (it carries the `@layer` line);
after that, layer order — not source order — decides the cascade. Then add one
component file per concept into `@layer components`.

```
css/
├── tokens.css      (a.k.a. _global.css)  ← load FIRST, carries the @layer line
├── reset.css
├── base.css
└── utilities.css
```

Loading, in any stack:

```html
<link rel="stylesheet" href="/css/tokens.css">
<link rel="stylesheet" href="/css/reset.css">
<link rel="stylesheet" href="/css/base.css">
<link rel="stylesheet" href="/css/utilities.css">
```

```css
/* …or a single bundler entry that imports each file, tokens first */
@import "tokens.css";
@import "reset.css";
@import "base.css";
@import "utilities.css";
```

---

## 1. tokens.css (_global.css)

The first line declares layer order; it must be the first declaration the browser
sees. The `:root` block is **unlayered** on purpose — unlayered styles rank above
all layered ones, so tokens are always available and never lose to a layered rule.

Stored in **two layers**: a bare `--lch-*` triplet (`L C H` — lightness %, chroma,
hue, no function wrapper), then a `--color-*` that wraps it in `oklch()`. Dark mode
re-points only the triplets, so every wrapped color follows for free.

```css
/* tokens.css — the very first declaration establishes the layer order */
@layer reset, base, components, modules, utilities;

:root {
  /* ---- OKLCH colors: Fixed (never change between themes) ---- */
  --lch-black: 0% 0 0;
  --lch-white: 100% 0 0;

  /* ---- Light-mode canvas / inverted-ink aliases ---- */
  --lch-canvas: var(--lch-white);
  --lch-ink-inverted: var(--lch-white);

  /* ---- Light-mode LCH scale: ink (near-neutral cool grey) ---- */
  --lch-ink-darkest: 26% 0.05 264;
  --lch-ink-darker:  40% 0.026 262;
  --lch-ink-dark:    56% 0.014 260;
  --lch-ink-medium:  66% 0.008 258;
  --lch-ink-light:   84% 0.005 256;
  --lch-ink-lighter: 92% 0.003 254;
  --lch-ink-lightest: 96% 0.002 252;

  /* ---- Accent hue families (dark/medium/light steps shown; expand from css-design-tokens) ---- */
  --lch-red-dark:    59% 0.19 38;
  --lch-red-medium:  66% 0.204 40;
  --lch-red-lighter: 92% 0.03 44;

  --lch-yellow-medium:  74% 0.184 70;
  --lch-yellow-lighter: 92% 0.076 90;

  --lch-green-dark:  55% 0.162 147;
  --lch-green-light: 83.92% 0.0772 145.06;

  --lch-blue-dark:     57.02% 0.1895 260.46;
  --lch-blue-medium:   66% 0.196 257.82;
  --lch-blue-light:    84.04% 0.0719 255.29;
  --lch-blue-lighter:  92% 0.026 254;
  --lch-blue-lightest: 96% 0.016 252;

  /* ---- Named colors (wrap the triplets) ---- */
  --color-black: oklch(var(--lch-black));
  --color-white: oklch(var(--lch-white));

  --color-ink:         oklch(var(--lch-ink-darkest));   /* "ink" unsuffixed = darkest = primary text */
  --color-ink-darkest: oklch(var(--lch-ink-darkest));
  --color-ink-darker:  oklch(var(--lch-ink-darker));
  --color-ink-dark:    oklch(var(--lch-ink-dark));
  --color-ink-medium:  oklch(var(--lch-ink-medium));
  --color-ink-light:   oklch(var(--lch-ink-light));
  --color-ink-lighter: oklch(var(--lch-ink-lighter));
  --color-ink-lightest: oklch(var(--lch-ink-lightest));
  --color-ink-inverted: oklch(var(--lch-ink-inverted));

  /* ---- Semantic abstractions: components read THESE, never raw --lch-* ---- */
  --color-canvas:        oklch(var(--lch-canvas));
  --color-link:          oklch(var(--lch-blue-dark));
  --color-link-50:       oklch(var(--lch-blue-dark) / 0.5);   /* alpha at point of use */
  --color-negative:      oklch(var(--lch-red-dark));
  --color-positive:      oklch(var(--lch-green-dark));
  --color-selected:      oklch(var(--lch-blue-lighter));
  --color-selected-light: oklch(var(--lch-blue-lightest));
  --color-selected-dark: oklch(var(--lch-blue-light));
  --color-highlight:     oklch(var(--lch-yellow-lighter));
  --color-marker:        oklch(var(--lch-red-medium));

  /* ---- Spacing: ch inline (tracks text measure), rem block (root rhythm) ---- */
  --inline-space: 1ch;
  --inline-space-half:   calc(var(--inline-space) / 2);
  --inline-space-double: calc(var(--inline-space) * 2);
  --block-space: 1rem;
  --block-space-half:   calc(var(--block-space) / 2);
  --block-space-double: calc(var(--block-space) * 2);

  /* ---- Text: system-font stacks only, no web-font loading, no FOUT ---- */
  --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
  --font-serif: ui-serif, serif;
  --font-mono: ui-monospace, monospace;

  /* ---- Type scale: all rem, keyed off the root font size ---- */
  --text-xx-small: 0.55rem;
  --text-x-small:  0.75rem;
  --text-small:    0.85rem;
  --text-normal:   1rem;
  --text-medium:   1.1rem;
  --text-large:    1.5rem;
  --text-x-large:  1.8rem;
  --text-xx-large: 2.5rem;

  /* ---- Borders ---- */
  --border: 1px solid var(--color-ink-lighter);

  /* ---- Shadow: four stacked black OKLCH layers at 5% alpha ---- */
  --shadow: 0 0 0 1px oklch(var(--lch-black) / 5%),
            0 0.2em 0.2em oklch(var(--lch-black) / 5%),
            0 0.4em 0.4em oklch(var(--lch-black) / 5%),
            0 0.8em 0.8em oklch(var(--lch-black) / 5%);

  /* ---- Component sizes ---- */
  --btn-size: 2.65em;
  --footer-height: 2.65rem;
  --tray-size: clamp(12rem, 25dvw, 24rem);

  /* ---- Focus rings (keyboard navigation), composed from sub-tokens ---- */
  --focus-ring-color: var(--color-link);
  --focus-ring-offset: 1px;
  --focus-ring-size: 2px;
  --focus-ring: var(--focus-ring-size) solid var(--focus-ring-color);

  /* ---- Dialog timing ---- */
  --dialog-duration: 150ms;

  /* ---- Easing functions (from https://easingwizard.com/) ---- */
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-out-overshoot: cubic-bezier(0.25, 1.75, 0.5, 1);
  --ease-out-overshoot-subtle: cubic-bezier(0.25, 1.25, 0.5, 1);

  /* ---- Layout (tokens composing tokens) ---- */
  --main-padding: clamp(var(--inline-space), 3vw, calc(var(--inline-space) * 3));
  --main-width: 1400px;

  /* ---- Named z-index scale: intention names, not magic numbers ---- */
  --z-popup: 10;
  --z-nav: 30;
  --z-flash: 40;
  --z-tooltip: 50;
  --z-bar: 60;
  --z-tray: 61;

  /* ---- Mobile bump: grow the small end of the type scale on narrow viewports ---- */
  @media (max-width: 639px) {
    --text-xx-small: 0.65rem;   /* up from 0.55rem */
    --text-x-small:  0.85rem;   /* up from 0.75rem */
    --text-small:    0.95rem;   /* up from 0.85rem */
    --text-normal:   1.1rem;    /* up from 1rem    */
    --text-medium:   1.2rem;    /* up from 1.1rem  */
    --text-large:    1.5rem;    /* unchanged       */
    --text-x-large:  1.8rem;    /* unchanged       */
    --text-xx-large: 2.5rem;    /* unchanged       */
  }
}

/* ---- Dark mode: flip the --lch-* triplets; zero component rules touched ---- */
/* 1. Explicit choice — beats the OS setting */
html[data-theme="dark"] {
  --lch-canvas: 20% 0.0195 232.58;     /* deep blue-grey, not pure black */
  --lch-ink-inverted: var(--lch-black);

  --lch-ink-darkest: 96.02% 0.0034 260;   /* now LIGHT — text on dark canvas */
  --lch-ink-darker:  86% 0.0061 260;
  --lch-ink-dark:    73.97% 0.009 260;
  --lch-ink-medium:  62% 0.0122 260;
  --lch-ink-light:   40% 0.0148 260;
  --lch-ink-lighter: 30% 0.0178 260;
  --lch-ink-lightest: 25% 0.0204 260;    /* now DARK — faint surface */

  --lch-blue-dark:     74% 0.1293 256;
  --lch-blue-medium:   62% 0.159 258;
  --lch-blue-light:    40% 0.094 260;
  --lch-blue-lighter:  30% 0.0452 262;
  --lch-blue-lightest: 25% 0.0318 264;

  --lch-red-dark:   73.95% 0.139 42;
  --lch-red-medium: 62% 0.154 40;

  --lch-green-dark:  73.99% 0.117 145;
  --lch-green-light: 40% 0.065 147;

  --shadow: 0 0 0 1px oklch(var(--lch-black) / 0.42),
    0 0.2em 1.6em -0.8em oklch(var(--lch-black) / 0.6),
    0 0.4em 2.4em -1em oklch(var(--lch-black) / 0.7),
    0 0.4em 0.8em -1.2em oklch(var(--lch-black) / 0.8),
    0 0.8em 1.2em -1.6em oklch(var(--lch-black) / 0.9),
    0 1.2em 1.6em -2em oklch(var(--lch-black) / 1);
}

/* 2. System fallback — only when NO explicit choice was made */
@media (prefers-color-scheme: dark) {
  html:not([data-theme]) {
    --lch-canvas: 20% 0.0195 232.58;
    --lch-ink-inverted: var(--lch-black);

    --lch-ink-darkest: 96.02% 0.0034 260;
    --lch-ink-darker:  86% 0.0061 260;
    --lch-ink-dark:    73.97% 0.009 260;
    --lch-ink-medium:  62% 0.0122 260;
    --lch-ink-light:   40% 0.0148 260;
    --lch-ink-lighter: 30% 0.0178 260;
    --lch-ink-lightest: 25% 0.0204 260;

    --lch-blue-dark:     74% 0.1293 256;
    --lch-blue-medium:   62% 0.159 258;
    --lch-blue-light:    40% 0.094 260;
    --lch-blue-lighter:  30% 0.0452 262;
    --lch-blue-lightest: 25% 0.0318 264;

    --lch-red-dark:   73.95% 0.139 42;
    --lch-red-medium: 62% 0.154 40;

    --lch-green-dark:  73.99% 0.117 145;
    --lch-green-light: 40% 0.065 147;

    --shadow: 0 0 0 1px oklch(var(--lch-black) / 0.42),
      0 0.2em 1.6em -0.8em oklch(var(--lch-black) / 0.6),
      0 0.4em 2.4em -1em oklch(var(--lch-black) / 0.7),
      0 0.4em 0.8em -1.2em oklch(var(--lch-black) / 0.8),
      0 0.8em 1.2em -1.6em oklch(var(--lch-black) / 0.9),
      0 1.2em 1.6em -2em oklch(var(--lch-black) / 1);
  }
}
```

> The dark-mode block above shows the **representative** families (ink, blue, red,
> green). The complete flip for every family — uncolor, yellow, lime, aqua, violet,
> purple, pink — lives in **css-design-tokens** (`references/color-system.md`). Add
> them following the same inversion: lightness climbs the other way, semantic names
> stay constant.

---

## 2. reset.css

A modern reset, opting into the `reset` layer. Logical properties throughout,
respects `prefers-reduced-motion`, and neutralizes the `<dialog>` / `<summary>`
defaults the rest of the system relies on.

```css
@layer reset {
  /* Box sizing rules */
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  /* Remove default margin */
  body, h1, h2, h3, h4, h5, h6 { margin: 0; }

  /* Prevent overflow of long words / URLs */
  p, li, h1, h2, h3, h4 { word-break: break-word; }

  html, body { overflow-x: clip; }

  body {
    min-height: 100dvh;
    font-family: sans-serif;
    font-size: 100%;
    line-height: 1.5;
    text-rendering: optimizeSpeed;
  }

  img { display: block; max-inline-size: 100%; }

  input, button, textarea, select { font: inherit; }
  button { cursor: pointer; }

  /* Strip the default disclosure marker so summary can be styled freely */
  summary {
    &::-webkit-details-marker { display: none; }
    &::marker { content: ""; }
  }

  /* Honor reduced-motion preferences */
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
    html { scroll-behavior: initial; }
  }

  /* Neutralize <dialog> defaults the component layer relies on */
  dialog {
    border: 0;
    padding: 0;

    &:where(:focus-visible):focus,
    &:where(:focus-visible):active { outline: 0; }
  }
}
```

---

## 3. base.css

Bare-element defaults in the `base` layer: typography on `body`, link styling on
classless `<a>`, the shared transition/focus/disabled treatment for the whole
interactive family via `:is()`, and theme-aware `::selection`. `:where()` keeps the
focus styling at zero specificity so utilities and components still override cleanly.

```css
@layer base {
  body {
    background-color: var(--color-canvas);
    color: var(--color-ink);
    font-family: var(--font-sans);
    font-size: var(--text-normal);
    line-height: 1.5;
  }

  /* Headings ride the type scale */
  h1 { font-size: var(--text-xx-large); }
  h2 { font-size: var(--text-x-large); }
  h3 { font-size: var(--text-large); }
  h4 { font-size: var(--text-medium); }

  code, pre, kbd { font-family: var(--font-mono); }

  a {
    text-decoration: none;

    &:not([class]) {
      color: var(--color-link);
      text-decoration: underline;
      text-decoration-skip-ink: auto;
    }
  }

  /* Shared treatment for the whole interactive family */
  :is(a, button, input, textarea, select, .switch, .btn) {
    transition: 100ms ease-out;
    transition-property: background-color, border-color, box-shadow, filter, outline;
    touch-action: manipulation;

    /* Keyboard navigation — zero-specificity so utilities still win */
    &:where(:focus-visible) {
      border-radius: 0.25ch;
      outline: var(--focus-ring-size) solid var(--focus-ring-color);
      outline-offset: var(--focus-ring-offset);
    }

    /* Default disabled styles */
    &:where([disabled]) {
      cursor: not-allowed;
      opacity: 0.5;
      pointer-events: none;
    }
  }

  /* Theme-aware text selection */
  ::selection {
    background: var(--color-selected);

    html[data-theme="dark"] & {
      background-color: var(--color-selected-dark);
    }

    @media (prefers-color-scheme: dark) {
      html:not([data-theme]) & {
        background-color: var(--color-selected-dark);
      }
    }
  }
}
```

---

## 4. utilities.css

Utilities are the **exception, not the foundation** — single-purpose layout glue.
The `utilities` layer is highest priority, so each utility already wins over any
component; wrapping in `:where()` keeps specificity at zero so it never needs
`!important`. Logical properties throughout, so spacing mirrors under RTL.

```css
@layer utilities {
  /* Display + flex/grid glue */
  :where(.flex) { display: flex; }
  :where(.grid) { display: grid; }
  :where(.inline-flex) { display: inline-flex; }
  :where(.gap) { gap: var(--inline-space); }
  :where(.gap-block) { gap: var(--block-space); }
  :where(.place-center) { place-items: center; }

  /* Padding (logical) */
  :where(.pad) { padding: var(--block-space) var(--inline-space); }
  :where(.pad-inline) { padding-inline: var(--inline-space); }
  :where(.pad-inline-start) { padding-inline-start: var(--inline-space); }
  :where(.pad-inline-end) { padding-inline-end: var(--inline-space); }
  :where(.pad-block) { padding-block: var(--block-space); }

  /* Margin (logical) */
  :where(.margin-block) { margin-block: var(--block-space); }
  :where(.margin-block-start) { margin-block-start: var(--block-space); }
  :where(.margin-inline-start) { margin-inline-start: var(--inline-space); }
  :where(.margin-auto) { margin: auto; }

  /* Type */
  :where(.txt-small) { font-size: var(--text-small); }
  :where(.txt-large) { font-size: var(--text-large); }
  :where(.txt-center) { text-align: center; }
  :where(.txt-start) { text-align: start; }

  /* Position */
  :where(.position-sticky) { position: sticky; inset: var(--inset, 0 auto auto auto); z-index: var(--z, 1); }

  /* Lists */
  :where(.list-style-none) { list-style: none; margin: 0 auto; padding: 0; }

  /* Capability-aware visibility — branch on input device, not screen width */
  :where(.hide) { display: none; }

  .hide-on-touch {
    @media (any-hover: none) { display: none; }
  }

  .show-on-touch {
    display: none;
    @media (any-hover: none) { display: unset; }
  }

  /* Screen-reader-only (accessible visually-hidden) */
  :where(.for-screen-reader) {
    position: absolute;
    inline-size: 1px;
    block-size: 1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
  }
}
```

---

## 5. Your first component file (the pattern to repeat)

Each new concept is **one flat file** opting into `@layer components`, built from
**local custom properties as its API.** A variant overrides only a variable — never
re-declares the rule. This is the template you copy for every component.

```css
/* buttons.css */
@layer components {
  .btn {
    align-items: center;
    background-color: var(--btn-background, var(--color-canvas));
    border-radius: var(--btn-border-radius, 2em);
    border: var(--btn-border-size, 1px) solid var(--btn-border-color, var(--color-ink-lighter));
    color: var(--btn-color, var(--color-ink));
    display: inline-flex;
    gap: 0.5em;
    justify-content: center;
    padding: var(--btn-padding, 0.5em 1.1em);
    transition: filter 100ms ease;

    @media (any-hover: hover) {
      &:where(:not(:active):hover) { filter: brightness(0.95); }
    }
  }

  /* Variants override ONLY variables — the rule above is never rewritten */
  .btn--reversed  { --btn-background: var(--color-ink); --btn-color: var(--color-ink-inverted); }
  .btn--negative  { --btn-background: var(--color-negative); --btn-color: var(--color-white); }
  .btn--borderless { --btn-border-color: transparent; }

  /* :has() lets the button adapt to its own contents — icon-only goes circular */
  .btn:where(:has(.for-screen-reader):has(img)) {
    --btn-border-radius: 50%;
    --btn-padding: 0;
    aspect-ratio: 1;
    block-size: var(--btn-size);
    inline-size: var(--btn-size);
    display: grid;
    place-items: center;
    > * { grid-area: 1/1; }
  }
}
```

```html
<button class="btn">Save</button>
<button class="btn btn--negative">Delete</button>
```

The HTML says **what the element is** (`btn btn--negative`), not how it looks. Change
`--btn-padding` once and every button updates. Add `--btn-background` for a variant
without touching the base rule. For the full component catalog — dialog with
`@starting-style`, the masked spinner, cards with `color-mix()` — see
**css-components** and **css-modern-features**.
