# Component Catalog — Extended, Verbatim

The full versions of the components summarized in `SKILL.md`, reproduced as they appear in the 37signals stylesheets (Campfire, Writebook, Fizzy). Every component lives inside `@layer components { ... }`, styles itself through component-scoped custom properties with inline fallbacks (`var(--x, default)`), and variants override **only variables** — never re-declaring layout. This is the "variables all the way down" architecture.

These are framework-agnostic CSS techniques: nothing here depends on a bundler, a templating language, or any JavaScript runtime. The only contract with the rest of the page is plain HTML and a handful of attributes (`[open]`, `[disabled]`, `[aria-busy]`, `[aria-checked]`, `data-theme="dark"`). Set those however you set attributes. Load these files however your stack loads CSS — a bundler entry that imports each file, `<link>` tags, or a framework's global stylesheet import.

Three conventions run through all of it:

- **Variables all the way down.** Every property is `var(--component-X, <inline default>)`. The component is usable standalone; the fallback is the default.
- **Variants flip variables, never layout.** Geometry is declared once on the base.
- **Logical properties + OKLCH + `color-mix()`** everywhere — `inline-size`, `padding-inline`, `margin-block`, `inset-*`, `oklch(var(--lch-*))`, derived palettes.

---

## 1. The Button — `.btn` (from `buttons.css`)

### Reusable principle

**Variables all the way down + variants flip variables.** The base `.btn` defines every property as `var(--btn-X, <default>)`. The inline default is the fallback, so the component renders standalone with zero config. Each variant class then sets only `--btn-*` variables — it never re-states `display`, `padding`, or `border-radius`. State (`:has(input:checked)`, `[disabled]`, busy form) also flips variables, not properties.

### Base component IN FULL

```css
@layer components {
  .btn {
    --icon-size: var(--btn-icon-size, 1.3em);
    --btn-border-radius: 99rem;
    --btn-hover-brightness: 0.9;

    align-items: center;
    background-color: var(--btn-background, var(--color-canvas));
    border-radius: var(--btn-border-radius);
    border: var(--btn-border-size, 1px) solid var(--btn-border-color, var(--color-ink-light));
    color: var(--btn-color, var(--color-ink));
    cursor: pointer;
    display: inline-flex;
    font-size: 1em;
    font-weight: var(--btn-font-weight, 600);
    gap: var(--btn-gap, 0.5em);
    justify-content: center;
    padding: var(--btn-padding, 0.5em 1.1em);
    pointer-events: auto;
    position: relative;
    transition: 100ms ease-out;
    transition-property: background-color, border, box-shadow, color, filter, opacity, scale;

    @media (any-hover: hover) {
      &:hover {
        filter: brightness(var(--btn-hover-brightness));
      }
    }

    html[data-theme="dark"] & {
      --btn-hover-brightness: 1.25;
    }

    @media (prefers-color-scheme: dark) {
      html:not([data-theme]) & {
        --btn-hover-brightness: 1.25;
      }
    }

    &[disabled],
    &:has([disabled]),
    [disabled] &[type=submit],
    &[type=submit]:disabled {
      cursor: not-allowed;
      opacity: 0.3;
      pointer-events: none;
    }

    form[aria-busy] &:disabled {
      position: relative;

      > * {
        visibility: hidden;
      }

      &::after {
        --mask: no-repeat radial-gradient(#000 68%,#0000 71%);
        --size: 1.25em;

        -webkit-mask: var(--mask), var(--mask), var(--mask);
        -webkit-mask-size: 28% 45%;
        animation: submitting 1s infinite linear;
        aspect-ratio: 8/5;
        background: currentColor;
        content: "";
        inline-size: var(--size);
        inset: 50%;
        margin-block: calc((var(--size) / 3) * -1);
        margin-inline: calc((var(--size) / 2) * -1);
        position: absolute;
      }
    }
  }
}
```

Note the **inline-fallback pattern**: `var(--btn-background, var(--color-canvas))` — component variable first, theme token as the fallback. The icon hooks into the same scheme: `--icon-size: var(--btn-icon-size, 1.3em)`. Dark mode and the busy spinner both work by setting variables (`--btn-hover-brightness`, `--mask`, `--size`), never by overriding the layout. The busy spinner is a masked `currentColor` block animated by sliding `mask-position` — no SVG, no GIF, no JS.

### Every variant — overriding ONLY variables

```css
  .btn--plain {
    --btn-background: transparent;
    --btn-border-radius: 0;
    --btn-border-size: 0;
    --btn-color: inherit;
    --btn-icon-size: 100%;
    --btn-padding: 0;
  }

  .btn--link {
    --btn-background: var(--color-link);
    --btn-border-color: var(--color-canvas);
    --btn-color: var(--color-ink-inverted);
    --focus-ring-color: var(--color-link);
  }

  .btn--negative {
    --btn-background: var(--color-negative);
    --btn-border-color: var(--color-negative);
    --btn-color: var(--color-ink-inverted);
    --focus-ring-color: var(--color-negative);
  }

  .btn--positive {
    --btn-background: var(--color-positive);
    --btn-border-color: var(--color-canvas);
    --btn-color: var(--color-ink-inverted);
    --focus-ring-color: var(--color-positive);
  }

  .btn--reversed {
    --btn-background: var(--color-ink);
    --btn-border-color: var(--color-canvas);
    --btn-color: var(--color-canvas);
    --focus-ring-color: var(--color-ink);
  }

  .btn--remove {
    --btn-icon-size: 0.7em;
  }

  /* Fake button used to help space things out */
  .btn--placeholder {
    pointer-events: none;
    visibility: hidden;
  }
```

`--circle` is special: it **also** matches icon-only buttons via `:has()` / `:where()` so the geometry is shared automatically. It still leans on variables (`--btn-padding`, `--icon-size`, `--btn-size`):

```css
  .btn--circle,
  .btn[aria-label]:where(:has(.icon)),
  .btn:where(:has(.for-screen-reader):has(.icon)) {
    --btn-padding: 0;
    --icon-size: 75%;

    aspect-ratio: 1;
    block-size: var(--btn-size);
    display: grid;
    inline-size: var(--btn-size);
    justify-content: normal; /* FF fix */
    place-items: center;

    > * {
      grid-area: 1/1;
    }
  }
```

`--success` is a pure-animation variant (no layout, only motion):

```css
  .btn--success {
    --success-timing-function: cubic-bezier(0.25, 1.25, 0.5, 1);
    animation: success 1s var(--success-timing-function);

    .icon {
      animation: zoom-fade 500ms var(--success-timing-function);
    }
  }
```

Responsive variants flip variables / visibility at breakpoints:

```css
  /* Make a normal button circular on mobile */
  @media (max-width: 639px) {
    .btn--circle-mobile {
      aspect-ratio: 1;
      padding: 0.5em;

      kbd,
      span:last-of-type {
        display: none;
      }
    }
  }

  @media (min-width: 640px) {
    .btn--circle-mobile .icon--mobile-only {
      display: none !important;
    }
  }

  .btn--back {
    --btn-border-size: 0;

    font-size: var(--text-medium);

    @media (max-width: 639px) {
      padding: 0.5em;

      strong, kbd {
        display: none;
      }
    }

    @media (min-width: 640px) {
      .icon--arrow-left {
        display: none;
      }
    }
  }
```

### Toggleable button — state via `:has()` flipping variables

A `.btn` wrapping a radio/checkbox becomes a self-styling toggle. The checked state sets variables; `:is()` groups the two input types:

```css
  .btn {
    &:has(input[type=radio], input[type=checkbox]) {
      position: relative;

      :is(input[type=radio], input[type=checkbox]) {
        appearance: none;
        border-radius: var(--btn-border-radius);
        cursor: pointer;
        display: flex;
        inset: 0;
        margin: 0;
        padding: 0;
        position: absolute;

        &:focus-visible {
          outline: none;
        }
      }

      .checked {
        display: none;
      }
    }

    &:has(input:checked)  {
      --btn-background: var(--color-ink);
      --btn-border-color: var(--color-ink);
      --btn-color: var(--color-ink-inverted);
      --focus-ring-color: var(--color-ink);

      .checked {
        display: block;
      }
    }

    &:has(input:focus-visible)  {
      outline: var(--focus-ring-size) solid var(--focus-ring-color);
      outline-offset: var(--focus-ring-offset);
    }
  }
```

### Button group — BEM `__group` element

The `__element` (group) re-sets child `--btn-*` variables to seam buttons together with logical-property radii (`border-end-start-radius`, etc.):

```css
  .btn__group {
    .btn {
      --btn-border-radius: 0;
      --radius: 0.3em;

      flex: 1 0 33%;
      inline-size: 100%;
      justify-content: center;
      white-space: nowrap;
    }

    form {
      flex: 1 1 0%;
    }

    :first-of-type .btn {
      border-end-start-radius: var(--radius);
      border-inline-end: 0;
      border-start-start-radius: var(--radius);
      padding-inline-end: 0.8em;
    }

    :last-of-type .btn {
      border-end-end-radius: var(--radius);
      border-inline-start: 0;
      border-start-end-radius: var(--radius);
      padding-inline-start: 0.8em;
    }

    span {
      inline-size: 100%;
    }
  }
```

### HTML usage

```html
<button class="btn">Save</button>
<button class="btn btn--negative">Delete</button>
<button class="btn btn--positive">Approve</button>
<button class="btn btn--reversed">Cancel</button>
<button class="btn btn--circle" aria-label="Close"><span class="icon icon--x"></span></button>

<!-- A toggle: the .btn re-themes itself when the inner input is checked -->
<label class="btn">
  <input type="checkbox" name="pinned">
  <span>Pin</span>
  <span class="checked icon icon--pin"></span>
</label>

<!-- A seamed group -->
<div class="btn__group">
  <button class="btn">Day</button>
  <button class="btn">Week</button>
  <button class="btn">Month</button>
</div>
```

---

## 2. The Card — `.card` (from `cards.css`, `card-perma.css`)

### Reusable principle

**One token, many derived colors via `color-mix()`.** The card takes a single `--card-color` and derives bg / content / text / border from it, each mixed toward canvas or ink by a fixed percentage. Set `--card-color` once (a board color, a "complete" color) and the whole card recolors coherently. Plus **BEM-like naming**: `.card__header`, `.card__title` (elements); `.card--notification` (modifier). Elements/modifiers add layout; the color system flows from variables.

### Base + the `color-mix()` derivation

```css
  .card {
    --avatar-size: 2.75em;
    --card-bg-color: color-mix(in srgb, var(--card-color) 4%, var(--color-canvas));
    --card-content-color: color-mix(in srgb, var(--card-color) 30%, var(--color-ink));
    --card-text-color: color-mix(in srgb, var(--card-color) 75%, var(--color-ink));
    --card-border: 1px solid color-mix(in srgb, var(--card-color) 33%, var(--color-ink-inverted));
    --card-header-space: 1ch;
    --card-padding-inline: var(--inline-space-double);
    --card-padding-block: var(--block-space);
    --border-color: transparent;
    --border-radius: 0.2em;
    --border-size: 0;

    aspect-ratio: var(--card-aspect-ratio, auto);
    background-color: var(--card-bg-color);
    border-radius: var(--border-radius);
    box-shadow: var(--shadow);
    display: flex;
    flex-direction: column;
    inline-size: 100%;
    padding: var(--card-padding-block) var(--card-padding-inline);
    position: relative;
    text-align: start;
    z-index: 1;

    html[data-theme="dark"] & {
      box-shadow: 0 0 0 1px var(--color-ink-lighter);
    }

    @media (prefers-color-scheme: dark) {
      html:not([data-theme]) & {
        box-shadow: 0 0 0 1px var(--color-ink-lighter);
      }
    }
  }
```

The four derived tokens (`--card-bg-color` 4%, `--card-content-color` 30%, `--card-text-color` 75%, `--card-border` 33%) are all `color-mix(in srgb, var(--card-color) N%, <base>)`. Increasing N pushes from a near-canvas tint to a near-ink tone — one knob recolors the entire card.

### BEM-like elements

The `__board` chip paints itself directly from `--card-color` and darkens on hover with another `color-mix()`:

```css
  .card__board {
    background-color: var(--card-color);
    border-radius: var(--border-radius) 0 var(--border-radius) 0;
    color: var(--color-ink-inverted);
    display: inline-flex;
    font-weight: 600;
    max-inline-size: 100%;
    min-inline-size: 0;
    padding-block: 0.25lh;
    padding-inline: var(--card-padding-inline) 1ch;

    &:has(.board-picker__button) {
      cursor: pointer;
      transition: background-color 100ms ease-out;

      @media (any-hover: hover) {
        &:hover {
          background-color: color-mix(in srgb, var(--card-color) 90%, var(--color-ink));
        }
      }
    }
  }
```

`.card__content` / `.card__title` / `.card__stages` consume the derived tokens:

```css
  .card__content {
    color: var(--card-content-color);
    contain: inline-size;
    flex: 2 1 auto;
    max-inline-size: 100%;
  }

  .card__title {
    --autosize-block-padding: 0 0.5ch;
    --input-border-radius: 0;
    --input-color: var(--card-content-color);
    --lines: 3;

    color: var(--card-content-color);
    font-size: var(--text-xx-large);
    font-weight: 900;
    line-height: 1.15;
    text-wrap: balance;
  }

  .card__stages {
    color: var(--card-text-color);
    display: flex;
    flex: 0 1 auto;
    flex-direction: column;
    gap: 2px;
    justify-self: end;
    max-inline-size: 20ch;
    padding-block: var(--block-space-half);
  }
```

The description scopes sibling buttons to the card color — a card-level theme reaching into the button component purely through variables:

```css
  .card__description {
    & ~ .btn.btn--reversed {
      --btn-background: var(--card-color);
      --btn-color: var(--color-ink-inverted);
    }

    & ~ .btn {
      --btn-border-color: var(--card-color);
      --btn-color: var(--card-color);
    }
  }
```

### `--modifier` flips the token

A "closed/postponed" card swaps its single `--card-color` token and the whole card recolors — derived bg/text/border follow automatically. `:is()` / `:has()` group the trigger conditions:

```css
  .card:has(.card__closed),
  .card:is(.card--postponed),
  .card-perma:has(.card__closed),
  .card-perma:has(.card--postponed) {
    --card-color: var(--color-card-complete) !important;

    .bubble {
      display: none;
    }
  }
```

The `--notification` modifier sets its own token + spacing variables, then re-themes child elements — still no layout duplication:

```css
  .card--notification {
    --card-color: var(--color-card-default);
    --card-padding-inline: 1ch;
    --card-padding-block: 1ch;

    background-color: var(--color-canvas);
    color: var(--color-ink);

    &.card--closed {
      --card-color: var(--color-card-complete) !important;
    }
  }
```

The unread indicator is a button re-skinned entirely through `--btn-*`:

```css
  .card__notification-unread-indicator {
    --btn-background: var(--color-marker);
    --btn-border-color: var(--color-canvas);
    --btn-color: var(--color-ink-inverted);
    --btn-icon-size: 0.5em;
    --btn-padding: 0;
    --btn-size: 1.6em;
  }
```

### Permalink container (`card-perma.css`)

A grid wrapper that introduces its own derived container color and overrides nested-component variables only:

```css
  .card-perma {
    --actions-block-inset: 1.5rem;
    --actions-inline-inset: 4rem;
    --color-container: color-mix(in srgb, var(--card-color) 33%, var(--color-canvas));
    --padding-inline: calc(var(--block-space-double) + var(--block-space));
    --padding-block: calc(var(--block-space-double) + var(--block-space-half));

    align-items: start;
    column-gap: var(--inline-space);
    display: grid;
    grid-template-areas:
      "notch-top notch-top notch-top"
      "actions-left card actions-right"
      "notch-bottom notch-bottom notch-bottom";
    grid-template-columns: 48px minmax(0, 1120px) 48px;
  }
```

Notch buttons recolor through `--btn-*` based on `--card-color` / `--color-container`:

```css
  .card-perma__notch--bottom {
    .btn:not(.popup__btn, .btn--plain, .btn--reversed) {
      --btn-background: var(--card-color);
      --btn-color: var(--color-ink-inverted);
    }

    .btn--reversed {
      --btn-background: var(--color-canvas);
      --btn-color: var(--card-color);
      --btn-border-color: var(--color-container);
    }
  }
```

### HTML usage

```html
<article class="card" style="--card-color: oklch(57% 0.19 260)">
  <span class="card__board">Marketing</span>
  <h3 class="card__title">Ship the redesign</h3>
  <div class="card__content">Final review with the team.</div>
  <ul class="card__stages"><li>In progress</li></ul>
</article>

<article class="card card--notification">…</article>
```

---

## 3. The Dialog — `:is(.dialog)` (from `dialog.css`)

### Reusable principle

**Entrance/exit animation with `@starting-style` + `transition-behavior: allow-discrete`** so a native top-layer element (and its `::backdrop`) animates in AND out — including the discrete `display` and `overlay` properties. No JS timers, no class juggling: the element transitions on `[open]`.

### IN FULL

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
      transform: scale(1);
      transition: var(--dialog-duration) allow-discrete;
      transition-property: display, opacity, overlay;
    }

    &[open] {
      opacity: 1;
      transform: scale(1);

      &::backdrop {
        opacity: 0.5;
      }
    }

    @starting-style {
      &[open] {
        opacity: 0;
        transform: scale(0.2);
      }

      &[open]::backdrop {
        opacity: 0;
      }
    }
  }
}
```

Key mechanics: `transition: <dur> allow-discrete` (the `allow-discrete` keyword on the shorthand enables transitioning discrete props), listing `display, overlay` in `transition-property`, and `@starting-style { &[open] { ... } }` supplying the "from" values for the first render in the top layer. The `::backdrop` animates in parallel. The popup component (below) reuses this exact recipe.

### HTML usage

```html
<dialog class="dialog">
  <form method="dialog">
    <p>Discard your changes?</p>
    <button class="btn btn--negative">Discard</button>
    <button class="btn">Keep editing</button>
  </form>
</dialog>
```

The only contract is the `[open]` attribute. Lock body scroll while it's open with `:has()` and no scroll-lock library: `html:has(.dialog[open]) { overflow: clip; }`.

---

## 4. Inputs, Switch, Bubble, Popup, Avatars, Blank-slates, Flash

### `.input` (from `inputs.css`)

**Variables all the way down**, same as `.btn` — every property has a `var(--input-X, default)`. Variants set only variables; containers re-zero them.

```css
  .input {
    accent-color: var(--input-accent-color, var(--color-ink));
    background-color: var(--input-background, transparent);
    border-radius: var(--input-border-radius, 0.5em);
    border: var(--input-border-size, 1px) solid var(--input-border-color, var(--color-ink-medium));
    color: var(--input-color, var(--color-ink));
    font-size: max(16px, 1em);
    inline-size: 100%;
    line-height: inherit;
    max-inline-size: 100%;
    padding: var(--input-padding, 0.5em 0.8em);
    resize: none;
  }
```

Variants flip variables (e.g. select with embedded SVG caret, swapped for dark):

```css
  .input--select {
    --input-border-radius: 2em;
    --input-padding: 0.5em 1.8em 0.5em 1.2em;
    --caret-icon: url("data:image/svg+xml,...fill='%23000'...");
    --caret-icon-dark: url("data:image/svg+xml,...fill='%23fff'...");

    -webkit-appearance: none;
    appearance: none;
    background-image: var(--caret-icon);
    background-size: 0.5em;
    background-position: center right 0.9em;
    background-repeat: no-repeat;
    text-align: start;

    html[data-theme="dark"] & {
      --caret-icon: var(--caret-icon-dark);
    }
  }
```

`.input--textarea` uses native `field-sizing: content` (no JS autosize):

```css
  .input--textarea {
    --input-padding: 0;
    min-block-size: calc(3lh + (2 * var(--input-padding)));

    @supports (field-sizing: content) {
      field-sizing: content;
      max-block-size: calc(3lh + (2 * var(--input-padding)));
      min-block-size: calc(1lh + (2 * var(--input-padding)));
    }
  }
```

A container that *acts like* an input re-zeros the nested `.input` via variables:

```css
  .input--actor {
    &:focus-within {
      --input-border-color: var(--color-selected-dark);
      outline: var(--focus-ring-size) solid var(--focus-ring-color);
    }

    .input {
      --input-padding: 0;
      --input-border-radius: 0;
      --input-background: transparent;
      --input-border-size: 0;
      inline-size: 100%;
      outline: 0;
    }
  }
```

### `.switch` — BEM toggle, state flips a variable (from `inputs.css`)

The visual track/thumb color is one variable `--switch-color`; checked/disabled sibling states reassign it; `:has(:focus-visible)` rings the `__btn`:

```css
  .switch {
    --switch-color: var(--color-ink-medium);
    --switch-hover-brightness: 0.9;

    block-size: 1.75em;
    border-radius: 2em;
    display: inline-flex;
    inline-size: 3em;
    position: relative;

    &:has(:focus-visible) {
      .switch__btn {
        outline: var(--focus-ring-size) solid var(--focus-ring-color);
      }
    }
  }

  .switch__btn {
    background-color: var(--switch-color);
    /* ... thumb is ::before, slides on checked ... */

    @media (any-hover: hover) {
      &:hover {
        background-color: color-mix(in srgb, var(--switch-color) 80%, var(--color-ink));
      }
    }

    .switch__input:checked + & {
      --switch-color: var(--color-link);

      &::before {
        transform: translateX(1.2em);
      }
    }

    .switch__input:disabled + & {
      --switch-color: var(--color-ink-medium);
      cursor: not-allowed;
      opacity: 0.5;
    }
  }
```

```html
<label class="switch">
  <input class="switch__input" type="checkbox" name="notify" hidden>
  <span class="switch__btn"></span>
</label>
```

### `.bubble` (from `bubble.css`)

**Token-derived color + container queries for self-scaling type.** One `--bubble-color` (falling back to `--card-color`, then an OKLCH blue) feeds a two-stop `color-mix()` radial gradient. `container-type: inline-size` lets the number scale with `cqi` units — the bubble is self-sizing:

```css
  .bubble {
    --bubble-color: var(--card-color, oklch(var(--lch-blue-medium)));
    --bubble-number-max: 26px;
    --bubble-shape: 54% 46% 61% 39% / 57% 49% 51% 43%;
    --bubble-rotate: 0deg;
    --bubble-size-default: 4rem;

    block-size: var(--bubble-size, var(--bubble-size-default));
    color: var(--card-content-color);
    container-type: inline-size;
    inline-size: var(--bubble-size, var(--bubble-size-default));

    &:before {
      background: radial-gradient(
        color-mix(in srgb, var(--bubble-color) 8%, var(--color-canvas)) 50%,
        color-mix(in srgb, var(--bubble-color) 48%, var(--color-canvas)) 100%
      );
      border-radius: var(--bubble-shape);
      transform: rotate(var(--bubble-rotate));
    }
  }

  .bubble__number {
    font-size: clamp(10px, 50cqi, var(--bubble-number-max)); /* scales to container */
  }
```

### `.popup` (from `popup.css`)

A native popover that **reuses the dialog entrance recipe** in `.popup--animated`, and uses `:has()` to hide empty sections automatically. Items re-skin `.btn` through `--btn-*`:

```css
  .popup {
    --btn-background: transparent;
    --panel-border-radius: 0.5em;
    --panel-padding: var(--block-space);
    --panel-size: auto;
    --popup-icon-size: 24px;
    --popup-item-padding-inline: 0.4rem;
    --popup-display: flex;

    &[open] {
      display: var(--popup-display);
    }
  }

  /* Hide lists when all the items within are hidden — pure :has() logic */
  .popup__section {
    &:not(:has(.popup__list)),
    &:not(:has(.popup__list > *)),
    &:has(.popup__item[hidden]):not(:has(.popup__item:not([hidden]))) {
      display: none;
    }
  }

  .popup__btn {
    --btn-border-radius: 0.3em;
    --btn-border-size: 0;
    justify-content: start;
    text-align: start;
  }

  /* Same @starting-style + allow-discrete recipe as the dialog */
  .popup--animated {
    opacity: 0;
    transform: scale(0.2) translateX(-50%);
    transform-origin: top left;
    transition: var(--dialog-duration) allow-discrete;
    transition-property: display, opacity, overlay, transform;

    &::backdrop {
      background-color: var(--color-always-black);
      opacity: 0;
      transition: var(--dialog-duration) allow-discrete;
      transition-property: display, opacity, overlay;
    }

    &[open] {
      opacity: 1;
      transform: scale(1) translateX(-50%);

      &::backdrop { opacity: 0.5; }
    }

    @starting-style {
      &[open] {
        opacity: 0;
        transform: scale(0.2) translateX(-50%);
      }
      &[open]::backdrop { opacity: 0; }
    }
  }
```

### `.avatar` (from `avatars.css`)

**Variable-driven square + `:is()` grouping** of `img` / `.icon` so either renders identically. Stacking/overlap of avatar groups is handled by the **card** (`cards.css`) via negative margins, not here:

```css
  .avatar {
    --avatar-border-radius: 50%;
    --avatar-size-default: 5ch;
    --btn-border-size: 0;

    aspect-ratio: 1;
    block-size: var(--avatar-size, var(--avatar-size-default));
    border-radius: var(--avatar-border-radius);
    display: grid;
    inline-size: var(--avatar-size, var(--avatar-size-default));
    place-items: center;

    :is(img, .icon) {
      aspect-ratio: 1;
      block-size: 100%;
      border-radius: var(--avatar-border-radius);
      grid-area: 1/1;
      inline-size: 100%;
      object-fit: cover;
    }
  }
```

The overlapping-group treatment lives in `cards.css` — avatars in an assignees group pull together with a negative logical margin (no per-avatar JS counting):

```css
  .card__meta-avatars--assignees {
    display: flex;
    margin-inline-start: var(--meta-spacer-inline);

    .avatar {
      margin-inline-end: calc(-1 * var(--meta-spacer-inline));
    }
  }
```

### `.blank-slate` (from `blank-slates.css`)

**Contextual `--modifier` + `:has()` presence logic.** Modifiers tint with `color-mix()` off `--card-color`, and ancestor `:has(...)` decides visibility — a list that already has real cards hides its empty-state:

```css
  .blank-slate--drag {
    box-shadow: none !important;
    color: color-mix(in srgb, var(--card-color) 70%, var(--color-canvas));

    .cards--considering & {
      background-color: var(--card-bg-color) !important;
    }
  }

  .blank-slate--empty {
    border: 2px dashed var(--color-ink-light);
    border-radius: 1ch;
    color: var(--color-ink-dark);
    margin-block-start: 5dvh;
    padding: 1ch 2ch;
    rotate: -3deg;
  }

  /* If the list contains a real (non-blank-slate) child, hide the empty filler */
  .cards__list:has(> :not(.blank-slate)) .card--hide-unless-empty {
    display: none;
  }
```

### `.flash` (from `flash.css`)

A fixed toast whose colors come from `var(--flash-X, default)` so a caller can recolor it without touching layout; the appear-then-fade is a CSS animation:

```css
  .flash {
    display: flex;
    inset-block-start: var(--block-space);
    inset-inline-start: 50%;
    position: fixed;
    transform: translate(-50%);
    z-index: var(--z-flash);
  }

  .flash__inner {
    animation: appear-then-fade 3s 300ms both;
    background-color: var(--flash-background, var(--color-ink));
    border-radius: 4em;
    color: var(--flash-color, var(--color-ink-inverted));
    padding: 0.7em 1.4em;
  }
```

---

## Cross-cutting principles (the takeaways)

1. **Variables all the way down.** Every base component (`.btn`, `.input`, `.card`, `.bubble`, `.flash`) declares properties as `var(--component-X, <inline default>)`. The component is usable standalone; the fallback is the default.
2. **Variants flip variables, never layout.** `.btn--negative`, `.btn--reversed`, `.input--select`, `.card--notification` set only `--*` tokens. Geometry is declared once.
3. **One token → many derived values via `color-mix()` / OKLCH.** `.card` derives bg/content/text/border from a single `--card-color`; `.bubble` mixes toward canvas/ink for gradients. Re-theme by changing one token.
4. **BEM-like `__element` / `--modifier` naming.** `.card__header`, `.btn__group`, `.switch__btn`, `.popup__item` (elements); `.card--notification`, `.btn--circle`, `.input--textarea` (modifiers).
5. **State via `:has()` / `:is()` / `:where()`, flipping variables.** Toggled buttons (`:has(input:checked)`), focus rings (`:has(:focus-visible)`), and empty-section hiding (`:not(:has(...))`) are pure CSS state — they reassign tokens or visibility, not the whole rule.
6. **Native modern motion.** `@starting-style` + `transition: <dur> allow-discrete` (with `display, overlay` in `transition-property`) animate top-layer elements in and out — shared verbatim by `.dialog` and `.popup--animated`. `field-sizing: content` replaces JS autosize.
7. **Logical properties + container queries throughout.** `padding-inline`, `margin-block`, `inset-*`, `block-size` / `inline-size`, and `cqi`-scaled type (`.bubble__number`) — direction-agnostic, self-scaling components.
