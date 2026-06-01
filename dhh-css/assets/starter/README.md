# dhh-css starter kit

A **copy-ready CSS design system** — 15 real `.css` files, framework-agnostic, zero build tools. Drop them into any project (plain HTML, React, Vue, Svelte, anything that loads a stylesheet) and you have a working, themeable, dark-mode-ready system before you write a single line of your own CSS.

Distilled from 37signals' Campfire / Writebook / Fizzy. Every value is real; nothing is invented.

## How to use

1. **Copy this folder** into your project's stylesheet directory.
2. **Load `_global.css` first.** It declares the cascade-layer order *and* defines every design token. Nothing else works correctly before it.
3. Load the rest in any order — they opt into named `@layer`s, so source order never decides who wins.

Load them however your stack loads CSS:

```html
<!-- _global.css MUST come first -->
<link rel="stylesheet" href="css/_global.css">
<link rel="stylesheet" href="css/reset.css">
<link rel="stylesheet" href="css/base.css">
<link rel="stylesheet" href="css/layout.css">
<link rel="stylesheet" href="css/utilities.css">
<link rel="stylesheet" href="css/buttons.css">
<!-- …the rest of the primitives… -->
```

```css
/* …or one bundler entry that imports each file, tokens first */
@import "_global.css";
@import "reset.css";
@import "base.css";
/* … */
```

## What's inside — two tiers

### Tier 1 — Foundation (the substrate; load order matters)

| File | Job |
| ---- | --- |
| `_global.css` | **The `@layer` declaration + every design token.** OKLCH color system (11 hue families × 7 steps), spacing, type scale (with a mobile bump), z-index scale, easings, focus ring, shadows. Dark mode by flipping the `--lch-*` triplets under `[data-theme="dark"]` *and* `prefers-color-scheme`. **Load first.** |
| `reset.css` | A modern reset — box-sizing, margin zeroing, media defaults, `prefers-reduced-motion` guard. |
| `base.css` | Bare-element typography & defaults (`body`, headings, `a`, `p`, lists, code, form controls) reading from tokens. |
| `layout.css` | A generic page shell — `.layout` grid (header / main / footer) with safe-area-aware padding. |
| `utilities.css` | Single-purpose helpers in the highest-priority layer: flex/gap/alignment, logical-property spacing, text utilities, `.hide`, `.for-screen-reader`. |

### Tier 2 — Reusable primitives (one file, many uses)

Each is **agnostic**: parametrized through CSS custom properties, selected by a semantic class, branched with `:has()` / attributes. The same `.btn` works in a toolbar, a form, a card, a dialog. Override the `--component-*` tokens to retheme; never edit the primitive.

| File | Primitive |
| ---- | --------- |
| `buttons.css` | `.btn` + variants (`--reversed` `--negative` `--positive` `--plain` `--circle`). The keystone — used everywhere. |
| `inputs.css` | Form fields — `input`, `textarea`, `select`, focus ring. |
| `icons.css` | A masked-icon **engine** (`.icon`, tinted by `currentColor`) + a few inline data-URI glyphs. Add your own in one line. |
| `animation.css` | A reusable `@keyframes` library (spin, pulse, fade, slide, shake) + a couple of animation utilities. |
| `spinners.css` | `.spinner` — a loading indicator built from CSS radial-gradient masks. No GIF, no SVG, no JS. |
| `dialog.css` | A generic `<dialog>` primitive with `@starting-style` enter/exit and `::backdrop` fade. |
| `avatars.css` | `.avatar` + a `:has()`-driven avatar-group layout that adapts to how many avatars it holds. |
| `bubble.css` | `.bubble` — a count chip / badge. |
| `dividers.css` | `.divider` and `.separator`. |
| `tooltips.css` | A `[data-tooltip]` tooltip — pure CSS, gated to real pointers. |

## Primitives vs feature components

This kit ships **primitives only** — the reusable substrate. It deliberately does **not** ship feature components (a kanban card, a nav bar, a comment thread), because those encode *your* product's domain.

When you build your own UI, follow the same rule the primitives follow:

- A **primitive** is reusable and domain-free. It belongs in `@layer components`, parametrized via `--component-*` tokens, named for what it *is* (`.btn`, `.card`), never for where it's used.
- A **feature component** composes primitives for one specific screen. Give it its own file in `@layer modules`, and let it set primitive tokens (`.checkout .btn { --btn-background: … }`) rather than restyling them.

If you find yourself hardcoding a color, a pixel value, or an app concept into one of these files — stop. Add a token, or make a new file. That's how the system stays coherent as it grows.

## Theming & dark mode

- Every color routes through a semantic token (`--color-ink`, `--color-canvas`, `--color-link`, …) built on raw `--lch-*` OKLCH triplets. Retheme by editing the triplets in `_global.css` — components never change.
- Dark mode is automatic: it follows the OS via `prefers-color-scheme`, or you can force it by setting `data-theme="dark"` on `<html>`. No component touches dark mode directly; the tokens flip and everything follows.

---

Want the full reasoning behind every pattern here? See the **dhh-css** skills: `css-design-system`, `css-design-tokens`, `css-components`, `css-modern-features`, `css-guardrails`.
