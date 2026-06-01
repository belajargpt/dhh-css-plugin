---
name: css-design-system
description: "Use when starting CSS from scratch — a blank project, a new app with no stylesheet, or an inconsistent mess to replace with a real system. Scaffolds the foundation in order: the @layer reset, base, components, modules, utilities declaration, a tokens file (_global.css / tokens.css with OKLCH --lch-* triplets, --inline-space/--block-space spacing, type scale), reset.css, base.css, and utilities.css. Covers flat one-file-per-concept organization, how to load CSS in any stack, the tokens → reset → base → component build order, and a starter checklist. Framework-agnostic 37signals-style vanilla CSS — no build tools, no Sass/PostCSS/Tailwind."
---

# CSS Design System

> Lay five files in one order — layers, tokens, reset, base, utilities — and you have a real design system before you write a single component.

This is the **blank-slate bootstrapper.** Point it at a project with no CSS, or an inconsistent pile of stylesheets you want to replace, and it stands up a coherent foundation you can build on for years. Everything here is **pure native CSS** — it reads identically in a single static `.html` file or any stack. No build step, no Sass, no PostCSS, no Tailwind.

Provenance: 37signals shipped three products — Campfire (chat), Writebook (publishing), Fizzy (kanban) — on roughly **14,000 lines of CSS across ~105 files with zero build tools.** The architecture below is mined verbatim from those stylesheets. It is cited only to credit where the patterns come from; nothing here requires any particular framework or templating language.

## The foundation: five files, one order

A design system is not a component library. It is the **substrate** every component reads from. You scaffold it in exactly this order, because each file depends on the one before it:

| # | File | Layer it owns | Job |
| - | ---- | ------------- | --- |
| 1 | *(the `@layer` line)* | — | Declares precedence **before any rule lands.** Lives at the very top of the tokens file. |
| 2 | `tokens.css` (`_global.css`) | unlayered `:root` | Colors, spacing, type scale, z-index — the vocabulary everything else speaks. |
| 3 | `reset.css` | `reset` | Neutralizes browser defaults so you build on flat ground. |
| 4 | `base.css` | `base` | Bare-element typography and defaults (`body`, `a`, headings, form controls). |
| 5 | `utilities.css` | `utilities` | A handful of single-purpose escape hatches — the highest-priority layer. |

After these five, you add **one component file per concept** (`buttons.css`, `dialog.css`, `cards.css`) into the `components` layer. That's the whole system.

## Step 1 — The @layer declaration

The single most important line in the codebase. `@layer` establishes precedence by **declaration order**: anything in a later layer beats an earlier one *regardless of selector specificity.* This is what kills `!important` wars and specificity escalation before they start.

```css
@layer reset, base, components, modules, utilities;
```

Layers, lowest → highest priority:

| Layer | Role |
| ----- | ---- |
| `reset` | The modern CSS reset (box-sizing, margin zeroing). |
| `base` | Element-level defaults / typography on bare tags. |
| `components` | Reusable component classes (buttons, dialog, cards). |
| `modules` | Larger feature/page modules built from components. |
| `utilities` | Single-purpose overrides — highest priority, always wins. |

This line must be the **first declaration the browser sees**, so the order is locked in before any rule is sorted into a layer. Put it at the top of your tokens file. Each subsequent file opts *into* its layer with `@layer <name> { … }`.

## Step 2 — tokens.css (a.k.a. _global.css)

The token layer is one `:root { … }` block of custom properties. Note it is deliberately **not** wrapped in a `@layer` — unlayered styles rank above all layered ones, so raw token definitions are always available and never lose to a layered rule. This is a trimmed starter; the full richer set is in `references/foundation.md`.

```css
@layer reset, base, components, modules, utilities;

:root {
  /* OKLCH colors: store the raw L C H triplet, then wrap it in oklch() */
  --lch-black: 0% 0 0;
  --lch-white: 100% 0 0;

  --lch-ink-darkest: 26% 0.05 264;    /* primary text */
  --lch-ink-medium:  66% 0.008 258;
  --lch-ink-lighter: 92% 0.003 254;   /* borders */
  --lch-canvas:      var(--lch-white);

  --lch-blue-dark:  57.02% 0.1895 260.46;
  --lch-red-dark:   59% 0.19 38;
  --lch-green-dark: 55% 0.162 147;

  /* Named + semantic colors — components read THESE, never raw --lch-* */
  --color-black:    oklch(var(--lch-black));
  --color-white:    oklch(var(--lch-white));
  --color-ink:      oklch(var(--lch-ink-darkest));
  --color-canvas:   oklch(var(--lch-canvas));
  --color-link:     oklch(var(--lch-blue-dark));
  --color-negative: oklch(var(--lch-red-dark));
  --color-positive: oklch(var(--lch-green-dark));

  /* Spacing: ch inline (tracks the text measure), rem block (root rhythm) */
  --inline-space: 1ch;
  --inline-space-half:   calc(var(--inline-space) / 2);
  --inline-space-double: calc(var(--inline-space) * 2);
  --block-space: 1rem;
  --block-space-half:   calc(var(--block-space) / 2);
  --block-space-double: calc(var(--block-space) * 2);

  /* Type — system fonts, no web-font loading, no FOUT */
  --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
  --font-mono: ui-monospace, monospace;

  --text-small:  0.85rem;
  --text-normal: 1rem;
  --text-medium: 1.1rem;
  --text-large:  1.5rem;
  --text-x-large: 1.8rem;

  /* Borders, focus ring, z-index */
  --border: 1px solid var(--color-ink-lighter);
  --color-ink-lighter: oklch(var(--lch-ink-lighter));
  --focus-ring-color: var(--color-link);
  --focus-ring-size: 2px;
  --focus-ring-offset: 1px;

  --z-popup: 10;
  --z-nav: 30;
  --z-tooltip: 50;
}
```

Dark mode is a one-layer flip: re-point the `--lch-*` triplets and every `oklch(var(--lch-…))` follows. Two trigger paths with identical bodies — explicit choice via `html[data-theme="dark"]`, system fallback via `@media (prefers-color-scheme: dark)` scoped to `html:not([data-theme])` so an explicit choice is never overridden. Full flip in `references/foundation.md`; deep dive in **css-design-tokens**.

## Step 3 — reset.css

A modern reset, opting into the `reset` layer. It uses logical properties (`max-inline-size`), respects `prefers-reduced-motion`, and neutralizes the `<dialog>`/`<summary>` defaults the rest of the system relies on.

```css
@layer reset {
  *, *::before, *::after { box-sizing: border-box; }

  body, h1, h2, h3, h4, h5, h6 { margin: 0; }

  p, li, h1, h2, h3, h4 { word-break: break-word; }   /* no overflow on long URLs */

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

  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
}
```

## Step 4 — base.css

Bare-element defaults in the `base` layer: typography on `body`, link styling on classless `<a>`, and shared transition/focus/disabled treatment for the whole interactive family via `:is()`. `:where()` keeps focus styling at zero specificity so utilities still win.

```css
@layer base {
  body {
    background-color: var(--color-canvas);
    color: var(--color-ink);
    font-family: var(--font-sans);
    font-size: var(--text-normal);
  }

  a {
    text-decoration: none;
    &:not([class]) {
      color: var(--color-link);
      text-decoration: underline;
      text-decoration-skip-ink: auto;
    }
  }

  :is(a, button, input, textarea, select, .btn) {
    transition: 100ms ease-out;
    transition-property: background-color, border-color, box-shadow, filter, outline;
    touch-action: manipulation;

    &:where(:focus-visible) {
      border-radius: 0.25ch;
      outline: var(--focus-ring-size) solid var(--focus-ring-color);
      outline-offset: var(--focus-ring-offset);
    }

    &:where([disabled]) {
      cursor: not-allowed;
      opacity: 0.5;
      pointer-events: none;
    }
  }

  ::selection { background: var(--color-selected, oklch(var(--lch-blue-light, 84% 0.0719 255))); }
}
```

## Step 5 — utilities.css

Utilities are the **exception, not the foundation** — single-purpose escape hatches for layout glue. Wrap each in `:where()` so it stays low-specificity; the `utilities` layer already guarantees it wins over components, so it never needs `!important`.

```css
@layer utilities {
  :where(.flex) { display: flex; }
  :where(.grid) { display: grid; }
  :where(.gap)  { gap: var(--inline-space); }

  :where(.pad)        { padding: var(--block-space) var(--inline-space); }
  :where(.pad-inline) { padding-inline: var(--inline-space); }
  :where(.margin-block) { margin-block: var(--block-space); }

  :where(.txt-large)  { font-size: var(--text-large); }
  :where(.txt-center) { text-align: center; }

  :where(.list-style-none) { list-style: none; margin: 0; padding: 0; }

  :where(.hide) { display: none; }
}
```

## Loading the CSS

Load these files **however your stack loads CSS** — a bundler entry that imports each file, `<link>` tags in document order, or your framework's global stylesheet import. The choice is yours; CSS itself only enforces **one** ordering rule:

> The `@layer reset, base, components, modules, utilities;` line must be the **first** declaration the browser parses.

Put that line at the top of `tokens.css` and load `tokens.css` first. After that, **file order does not matter** — `@layer` decides the winner, not source position. You could load `buttons.css` before `reset.css` and the cascade would still resolve correctly, because every rule is sorted into its declared layer. That is the whole point of layers: source order stops being load-bearing.

```html
<!-- Plain static HTML — link tags, tokens first -->
<link rel="stylesheet" href="/css/tokens.css">
<link rel="stylesheet" href="/css/reset.css">
<link rel="stylesheet" href="/css/base.css">
<link rel="stylesheet" href="/css/utilities.css">
<link rel="stylesheet" href="/css/buttons.css">
```

```css
/* A bundler entry — one file that imports the rest, tokens first */
@import "tokens.css";
@import "reset.css";
@import "base.css";
@import "utilities.css";
@import "buttons.css";
```

## Flat file organization

**One file per concept. No subdirectories, no partials, no import trees.** Each file is named exactly what it does, so you find styles by guessing the obvious filename.

```
css/
├── tokens.css        (or _global.css — colors, spacing, type, z-index)
├── reset.css
├── base.css
├── utilities.css
├── buttons.css
├── inputs.css
├── dialog.css
├── cards.css
└── [concept].css     (one new flat file per component)
```

That is it. No `components/` folder, no `_mixins`, no five-level deep `@import` graph. A 14,000-line system lives comfortably in ~105 flat files this way. When you need a new component, you add **one file**, opt it into `@layer components`, and you are done.

## Order of operations when building a UI

Build the substrate bottom-up, then add components on top:

1. **Tokens** — define the vocabulary first. You cannot reference `--color-ink` before it exists.
2. **Reset** — flatten browser defaults so you build on level ground.
3. **Base** — set bare-element typography and the shared interactive treatment.
4. **Utilities** — add layout escape hatches as you discover you need them (not upfront).
5. **One component file per concept** — `buttons.css`, `dialog.css`, etc., each into `@layer components`, each consuming tokens through local custom-property APIs.

Never skip ahead. A button hardcoded with `#3b82f6` before the token layer exists is a button you will rewrite. Define the token, then consume it.

## Design system starter checklist

- [ ] `@layer reset, base, components, modules, utilities;` is the **first** line of `tokens.css`.
- [ ] `tokens.css` defines `--lch-*` triplets, `--color-*` semantics, `--inline-space`/`--block-space`, a type scale, and a named `--z-*` scale.
- [ ] `:root` token block is **unlayered** (sits above all layers).
- [ ] `reset.css` is wrapped in `@layer reset` and handles box-sizing, margin zeroing, `prefers-reduced-motion`.
- [ ] `base.css` is wrapped in `@layer base`; `body` reads `--color-canvas` / `--color-ink` / `--font-sans`.
- [ ] Shared `:focus-visible` ring lives in `base`, composed from `--focus-ring-*` tokens.
- [ ] `utilities.css` is wrapped in `@layer utilities`; each utility uses `:where()` for zero specificity.
- [ ] Spacing consumed via **logical properties** (`padding-inline`, `margin-block`), never `padding-left`/`margin-top`.
- [ ] Dark mode is a `--lch-*` flip in `html[data-theme="dark"]` + a `prefers-color-scheme` fallback guarded by `:not([data-theme])`.
- [ ] Files are **flat** — one per concept, no subdirectories or partials.
- [ ] Components are added one file at a time into `@layer components`.

## Anti-Patterns

| Anti-pattern | Why it hurts | Do instead |
| ------------ | ------------ | ---------- |
| No `@layer` line; fighting specificity with `!important` | Cascade is unreadable; every fix spawns the next | Declare the layer order first; let layers decide winners |
| `@layer` line buried below other rules | Layer order isn't established before rules land | Make it the first declaration in `tokens.css` |
| Wrapping `:root` tokens in a layer | A layered rule can now beat your tokens | Leave `:root` unlayered — it should rank above everything |
| `components/buttons/_primary.scss` partial trees | Find-by-filename breaks; needs a build step | Flat `buttons.css`, one file per concept |
| Building components before tokens exist | Hardcoded values you'll rewrite | Tokens → reset → base → components, in order |
| `padding-left` / `margin-top` in the foundation | Breaks under RTL / vertical writing | `padding-inline` / `margin-block` |
| Utility-first soup as the *foundation* | HTML stops saying what things ARE | Semantic components; utilities are escape hatches only |
| Reaching for Sass/PostCSS to get nesting/variables | A build step for what's native now | Native nesting + custom properties |

## When to use this skill

Use **css-design-system** when there is **nothing to build on yet** — a blank project, a fresh app with no stylesheet, or a tangle of inconsistent CSS you're replacing wholesale. It gives you the substrate. Once the five foundation files exist, switch to the focused siblings: **css-design-tokens** to expand the palette and theming, **css-components** to build each component file, **css-modern-features** for the advanced toolkit, and **css-guardrails** to review what you wrote.

## Related Skills

- **css-design-tokens** — the full OKLCH color system, every hue family, dark-mode flip, spacing/type/z-index scales that fill out the `tokens.css` you scaffold here.
- **css-components** — how to write each `@layer components` file: buttons, dialog, cards, consuming tokens via local custom-property APIs.
- **css-modern-features** — `:has()`, `@starting-style`, container queries, `color-mix()`, masks, native nesting — the modern toolkit you build components from.
- **css-guardrails** — the review pass that keeps the foundation clean: no `!important`, no hardcoded values, no magic numbers.
