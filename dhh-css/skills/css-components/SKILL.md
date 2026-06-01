---
name: css-components
description: "Use when building UI components in pure CSS — semantic component classes (.btn, .card, .dialog, .input, .switch) instead of utility-class soup, with component-level custom properties and inline fallbacks var(--btn-bg, default), BEM-like __element / --modifier naming, variants that flip only variables, :is()/:where()/:has() to group selectors and tame specificity, color-mix() to derive a whole bg/text/border palette from one --color token, container queries so a component responds to its container not the viewport, and co-located hover (any-hover: hover), dark mode, and responsive rules. Framework-agnostic 37signals-style vanilla CSS — zero build step, equally usable in plain static HTML or any stack."
---

# CSS Components

> Name the thing, not the look. A component is a small API: a bag of `--vars` with sane defaults, where a variant overrides only what changes.

A component is `class="btn"`, not `class="inline-flex items-center gap-2 px-4 py-2 rounded-full border bg-white text-gray-900 hover:brightness-95"`. The first reads as *what the element is*; the second is the element's entire stylesheet pasted into the markup. 37signals built Campfire, Writebook, and Fizzy this way — semantic component classes, **zero build tools**, no Sass/PostCSS/Tailwind. Every example below is real, lifted from those stylesheets.

Load the CSS however your stack loads CSS — a bundler entry that imports each file, `<link>` tags, or your framework's global stylesheet import. The only contract with the rest of the page is plain HTML and a few attributes (`[open]`, `[disabled]`, `data-theme="dark"`). A component is just a class name applied to semantic HTML.

## Why semantic classes over utility soup

| | Utility soup | Semantic component |
| --- | --- | --- |
| **Read the HTML** | A wall of class names; you reverse-engineer the look to learn what it is | `class="btn btn--negative"` — it *is* a negative button |
| **Make a change** | Edit every element that pasted the same classes | Edit `--btn-padding` once; every `.btn` updates |
| **Add a variant** | Copy the base classes, swap a few | Add `.btn--circle`; redefine nothing |
| **Co-locate rules** | Hover/dark/responsive scattered across breakpoint prefixes | `@media any-hover`, dark mode, container query all live *inside* the component block |

The thesis: HTML stays declarative (it says what it is), changes **cascade** from one place, variants **compose** without re-stating geometry, and a component's media queries **live with the component**.

**You are building reusable primitives.** A well-made component is *agnostic* — one `.btn` or `.card` file serving a toolbar, a form, a dialog, and a list without modification. That reusability is not a separate technique; it's the payoff of the two patterns below (parametrize via variables, select via semantic class + state). Keep components domain-free: the moment one names an app concept (`.checkout`, `.kanban-column`), it has become a *feature component* and belongs in `@layer modules`, composing primitives by setting their tokens. See the primitive-vs-feature rule in **css-guardrails**, and the ready-made primitives in this plugin's `assets/starter/`.

## Variables all the way down

A base component declares every visual property as `var(--component-X, <inline default>)`. The inline fallback is the default, so the component renders standalone with zero config. Callers and variants then override *only the variable that changes* — never the property, never the layout.

```css
.btn {
  align-items: center;
  background-color: var(--btn-background, var(--color-canvas));
  border-radius: var(--btn-border-radius);
  border: var(--btn-border-size, 1px) solid var(--btn-border-color, var(--color-ink-light));
  color: var(--btn-color, var(--color-ink));
  display: inline-flex;
  gap: 0.5em;
  justify-content: center;
  padding: var(--btn-padding, 0.5em 1.1em);
}
```

This is the **inline-fallback pattern**: component variable first, theme token as the fallback. The boilerplate-heavy alternative declared every `--btn-*` at the top of the rule and then used it; inline fallbacks collapse that to one line per property *and* document the default right where it's consumed. From 37signals: "This makes it very clear what's changed by these variants."

DRY shared values still belong at the root, where multiple components compose them:

```css
:root { --btn-size: 2.65em; }
body {
  --footer-height: calc((var(--block-space)) + var(--btn-size) + var(--block-space));
}
```

## Variants flip variables only

A variant is a class that sets `--component-*` tokens and **nothing else**. No re-stated `display`, no re-stated `padding`. Geometry is declared once on the base; the variant is a tiny, readable, magic-number-free mini-API.

```css
.btn--reversed  { --btn-background: var(--color-ink); }
.btn--negative  { --btn-background: var(--color-negative); }
:is(.btn--reversed, .btn--negative) { --btn-color: var(--color-ink-inverted); }
.btn--borderless { --btn-border-color: transparent; }
```

`:is(.btn--reversed, .btn--negative)` groups the two variants that share a text color into one rule. Because a variant touches only variables, it stacks cleanly with any geometry variant (`.btn--circle`, `.btn__group .btn`): the geometry variant owns layout, the color variant owns color, neither steps on the other.

## BEM-like naming + :is()/:where() for specificity

Two suffixes, mined throughout Fizzy:

- **`__element`** — a structural part owned by the component: `.card__header`, `.card__title`, `.btn__group`, `.switch__btn`, `.popup__item`.
- **`--modifier`** — a variant of the component: `.card--notification`, `.btn--circle`, `.input--textarea`.

`:is()` and `:where()` keep the selectors flat and the specificity predictable. `:is(a, b, c)` groups selectors and takes the specificity of its *most specific* argument; `:where(...)` is identical but contributes **zero** specificity — the override valve. Use `:where()` to attach state to a component without raising its specificity, so a later variant or utility still wins:

```css
/* Shared transitions + focus ring for a whole family, one rule */
:is(a, button, input, textarea, .switch, .btn) {
  transition: 100ms ease-out;
  transition-property: background-color, border-color, box-shadow, filter, outline;

  &:where(:focus-visible) {
    outline: var(--focus-ring-size) solid var(--focus-ring-color);
    outline-offset: var(--focus-ring-offset);
  }
}
```

State driven by `:has()` flips variables too — a button wrapping a checked input re-themes itself with no JS and no toggled ancestor class:

```css
.btn:has(input:checked) {
  --btn-background: var(--color-ink);
  --btn-border-color: var(--color-ink);
  --btn-color: var(--color-ink-inverted);
}
```

Wrap components in the `components` cascade layer so utilities always win without `!important`:

```css
@layer reset, base, components, modules, utilities;
@layer components { .btn { /* always lower priority than utilities */ } }
```

## color-mix(): one token → a whole palette

When a component needs a coordinated background / content / text / border set, don't hand-pick colors — derive them from a single seed with `color-mix()`. Each property mixes the seed toward `--color-canvas` (lighter) or `--color-ink` (darker) by a fixed percentage. The card takes one `--card-color` and the whole card recolors coherently:

```css
.card {
  --card-color: oklch(var(--lch-blue-dark));
  --card-bg-color:      color-mix(in srgb, var(--card-color) 4%,  var(--color-canvas));
  --card-content-color: color-mix(in srgb, var(--card-color) 30%, var(--color-ink));
  --card-text-color:    color-mix(in srgb, var(--card-color) 75%, var(--color-ink));
  --card-border: 1px solid color-mix(in srgb, var(--card-color) 33%, var(--color-ink-inverted));
}
```

Increasing the percentage walks the result from a near-canvas tint (4%) to a near-ink tone (75%) — **one knob recolors the entire card**. Change `--card-color` to any hue and bg/content/text/border re-derive in harmony. And because the seeds (`--color-canvas`, `--color-ink`) themselves flip in dark mode, the derived palette adapts to dark mode for free. A hover state is just another mix off the same token:

```css
.card__board {
  background-color: var(--card-color);

  @media (any-hover: hover) {
    &:hover { background-color: color-mix(in srgb, var(--card-color) 90%, var(--color-ink)); }
  }
}
```

## Container queries: respond to the container, not the viewport

A component should size itself to the box it lives in, not to the screen. Make the component a query container with `container-type: inline-size`, then size contents in `cqi` (1% of container inline size). The number inside a bubble scales with its own container — drop the bubble in a wide card or a narrow chip and it just works:

```css
.bubble {
  --bubble-number-max: 26px;
  container-type: inline-size;
  inline-size: var(--bubble-size, var(--bubble-size-default));
}

.bubble__number {
  font-size: clamp(10px, 50cqi, var(--bubble-number-max)); /* scales to container */
}
```

This is true component-level responsiveness: the same component behaves identically in a wide desktop column or a narrow mobile one, with no viewport breakpoints to juggle, because it measures its own container.

## Co-locate hover, dark mode, and responsive rules

Everything about a component lives inside its block — native nesting, no preprocessor. Hover gates on `any-hover: hover` so touch users never get stuck-on hover states. Dark mode reads two attribute contracts (an explicit `data-theme` and the system fallback). Responsive rules nest right where the component is defined.

```css
.btn {
  --btn-hover-brightness: 0.9;

  /* hover only where the device can actually hover */
  @media (any-hover: hover) {
    &:hover { filter: brightness(var(--btn-hover-brightness)); }
  }

  /* dark mode: explicit choice */
  html[data-theme="dark"] & { --btn-hover-brightness: 1.25; }

  /* dark mode: system fallback, only when no explicit choice was made */
  @media (prefers-color-scheme: dark) {
    html:not([data-theme]) & { --btn-hover-brightness: 1.25; }
  }
}
```

The `:not([data-theme])` guard is the crux of dark mode: the system query applies **only when no explicit theme is set**, so a user who chose light is never overridden by a dark OS setting. The CSS contract is one attribute — *when `<html>` has `data-theme="dark"`, the dark values apply* — whatever sets that attribute is outside the CSS. Capability queries also branch on the *input device* to reveal hover-only controls:

```css
@media (any-hover: hover) and (pointer: fine)  { .row .controls { opacity: 0; } .row:hover .controls { opacity: 1; } }
@media (any-hover: none)  and (pointer: coarse) { .show-on-touch { display: unset; } }
```

## Worked example: the button

The base declares every property through a fallback variable; variants below flip only tokens. (Fuller version, including the `:has()` toggle and `__group`, in `references/component-catalog.md`.)

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
    font-weight: var(--btn-font-weight, 600);
    gap: var(--btn-gap, 0.5em);
    justify-content: center;
    padding: var(--btn-padding, 0.5em 1.1em);
    position: relative;
    transition: 100ms ease-out;
    transition-property: background-color, border, box-shadow, color, filter, opacity, scale;

    @media (any-hover: hover) {
      &:hover { filter: brightness(var(--btn-hover-brightness)); }
    }

    &[disabled], &:has([disabled]) {
      cursor: not-allowed;
      opacity: 0.3;
      pointer-events: none;
    }
  }

  /* Variants flip ONLY variables — no re-stated geometry */
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
  }

  .btn--plain {
    --btn-background: transparent;
    --btn-border-radius: 0;
    --btn-border-size: 0;
    --btn-color: inherit;
    --btn-padding: 0;
  }

  /* Geometry variant — also matches icon-only buttons via :has(), so the shape is shared */
  .btn--circle,
  .btn[aria-label]:where(:has(.icon)) {
    --btn-padding: 0;
    --icon-size: 75%;
    aspect-ratio: 1;
    block-size: var(--btn-size);
    display: grid;
    inline-size: var(--btn-size);
    place-items: center;
    > * { grid-area: 1/1; }
  }
}
```

```html
<!-- Plain, semantic HTML. No framework, no utility classes. -->
<button class="btn">Save</button>
<button class="btn btn--negative">Delete</button>
<button class="btn btn--positive">Approve</button>
<button class="btn btn--circle" aria-label="Close"><span class="icon icon--x"></span></button>
```

## Worked example: the card (color-mix derivation)

One `--card-color` seeds the whole surface; `__element` parts consume the derived tokens; a `--modifier` swaps the seed and the card recolors top to bottom.

```css
@layer components {
  .card {
    --card-bg-color:      color-mix(in srgb, var(--card-color) 4%,  var(--color-canvas));
    --card-content-color: color-mix(in srgb, var(--card-color) 30%, var(--color-ink));
    --card-text-color:    color-mix(in srgb, var(--card-color) 75%, var(--color-ink));
    --card-border: 1px solid color-mix(in srgb, var(--card-color) 33%, var(--color-ink-inverted));
    --card-padding-inline: var(--inline-space-double);
    --card-padding-block: var(--block-space);

    background-color: var(--card-bg-color);
    border-radius: 0.2em;
    box-shadow: var(--shadow);
    display: flex;
    flex-direction: column;
    inline-size: 100%;
    padding: var(--card-padding-block) var(--card-padding-inline);
    position: relative;
  }

  .card__title   { color: var(--card-content-color); font-weight: 900; text-wrap: balance; }
  .card__content { color: var(--card-content-color); }
  .card__stages  { color: var(--card-text-color); }

  /* --modifier swaps the single token; derived bg/text/border follow automatically */
  .card--notification {
    --card-color: var(--color-card-default);
    --card-padding-inline: 1ch;
    --card-padding-block: 1ch;

    &.card--closed { --card-color: var(--color-card-complete); }
  }
}
```

```html
<article class="card" style="--card-color: oklch(57% 0.19 260)">
  <h3 class="card__title">Ship the redesign</h3>
  <p class="card__content">Final review with the team.</p>
  <ul class="card__stages"><li>In progress</li></ul>
</article>
```

## Worked example: the dialog (@starting-style entrance)

A native top-layer element that animates **in and out** with no JS class toggling. `transition: <dur> allow-discrete` lets the discrete `display`/`overlay` properties participate; `@starting-style` supplies the first-frame "from" values so the open animation plays the very first time the element appears. The `::backdrop` animates in parallel.

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

```html
<dialog class="dialog">
  <form method="dialog">
    <p>Discard your changes?</p>
    <button class="btn btn--negative">Discard</button>
    <button class="btn">Keep editing</button>
  </form>
</dialog>
```

The only contract is the `[open]` attribute — the dialog animates on its presence, no `requestAnimationFrame`, no `transitionend` listener. The popup component reuses this exact recipe.

## Worked example: the input / field

Same architecture as `.btn` — every property a fallback variable, variants flip tokens, native features replace JS. `.input--textarea` uses `field-sizing: content` for no-JS autosize; a container that *acts like* an input re-zeros the nested `.input` through variables.

```css
@layer components {
  .input {
    accent-color: var(--input-accent-color, var(--color-ink));
    background-color: var(--input-background, transparent);
    border-radius: var(--input-border-radius, 0.5em);
    border: var(--input-border-size, 1px) solid var(--input-border-color, var(--color-ink-medium));
    color: var(--input-color, var(--color-ink));
    font-size: max(16px, 1em);   /* no zoom-on-focus on iOS */
    inline-size: 100%;
    padding: var(--input-padding, 0.5em 0.8em);
    resize: none;
  }

  .input--textarea {
    --input-padding: 0;
    @supports (field-sizing: content) {
      field-sizing: content;          /* grows with content, no JS */
      max-block-size: calc(3lh + (2 * var(--input-padding)));
      min-block-size: calc(1lh + (2 * var(--input-padding)));
    }
  }

  /* A wrapper that re-zeros a nested .input purely through variables */
  .input--actor {
    &:focus-within {
      --input-border-color: var(--color-selected-dark);
      outline: var(--focus-ring-size) solid var(--focus-ring-color);
    }
    .input {
      --input-padding: 0;
      --input-border-size: 0;
      --input-background: transparent;
    }
  }
}
```

```html
<input class="input" type="text" name="title" placeholder="Title">
<textarea class="input input--textarea" name="body"></textarea>
```

## Anti-Patterns

| Anti-pattern | Why it hurts | Do instead |
| --- | --- | --- |
| `class="flex items-center gap-2 px-4 ..."` for a button | HTML stops saying what it is; every instance re-pastes the look | `class="btn"`; one semantic class |
| A variant that re-states layout (`.btn--negative { display: inline-flex; padding: ...; background: red }`) | Geometry duplicated; drifts from the base | Variant sets `--btn-*` tokens only |
| Hardcoding `background: red` in a variant | Bypasses theming and dark mode | `--btn-background: var(--color-negative)` |
| Hand-picking bg/text/border per card color | Three colors to maintain per hue, easy to drift | Derive all three with `color-mix()` from one `--card-color` |
| `width` / `padding-left` / `margin-top` | Breaks under RTL / vertical writing modes | `inline-size` / `padding-inline` / `margin-block` |
| `:hover` with no `any-hover` guard | Touch devices get stuck-on hover states | Wrap in `@media (any-hover: hover)` |
| JS class toggling for enter/exit animation | Timers, listeners, leftover classes | `@starting-style` + `allow-discrete` on the `[open]` contract |
| Viewport `@media` to resize a component | Same component, two layouts to maintain | `container-type` + `cqi`, respond to the container |
| `!important` to beat a utility | Specificity war | Put components in `@layer components`; utilities win by layer order |

## When to use

- **Reach for a semantic component class** the moment a pattern appears twice, or the moment the markup needs to read as *what it is* (a form, a card grid, a toolbar).
- **Expose a `--var` only when a caller or variant will plausibly override it.** Don't pre-emptively variable-ize every property; variable-ize the ones that define the component's API (color, padding, radius, size).
- **Let a single token seed derived colors** (`color-mix()`) when a component is categorically recolored (board colors, status colors). Otherwise a plain semantic color is fine.
- **Keep one file per component**, named exactly what it is (`buttons.css`, `cards.css`, `dialog.css`, `inputs.css`) — no subdirectories, no partial trees.

## Related Skills

- **css-design-system** — the no-build file organization and `@layer reset, base, components, modules, utilities` architecture these components live inside.
- **css-design-tokens** — the OKLCH `--lch-*` / `--color-*` and spacing tokens (`--inline-space`, `--block-space`) that components consume as fallbacks.
- **css-modern-features** — `:has()`, `:is()`/`:where()`, `@starting-style`, container queries, `color-mix()`, masks, and `field-sizing` in depth.
- **css-guardrails** — the review rules that keep variants flipping variables (not layout) and components reading only the semantic token layer.
