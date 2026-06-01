# dhh-css — CSS Design System Conventions

> *"While the industry sprints toward increasingly complex toolchains, 37signals is walking calmly in the other direction."*

This plugin teaches Claude Code to write **a real CSS design system the 37signals way** — modern, native, framework-agnostic, with **zero build tools**. The context below loads every session so every stylesheet you scaffold, every token you define, and every component you style stays consistent with one philosophy.

This is **pure CSS**. It drops into plain static HTML, React, Vue, Svelte, or anything that loads a stylesheet — load the files however your stack loads CSS (a bundler entry that imports each file, `<link>` tags, or your framework's global stylesheet import). There is no asset pipeline, no preprocessor, no framework requirement anywhere in this plugin.

**Real files ship with the plugin.** A copy-ready starter kit of 15 framework-agnostic `.css` files lives at `assets/starter/` — a foundation (`_global.css` tokens + `reset`/`base`/`layout`/`utilities`) plus reusable primitives (`buttons`, `inputs`, `icons`, `animation`, `spinners`, `dialog`, `avatars`, `bubble`, `dividers`, `tooltips`). To bootstrap a blank project, **copy those files** rather than regenerating them; load `_global.css` first. The starter ships *primitives only* — build feature components on top in `@layer modules` by setting primitive tokens, never by restyling them.

## Core Philosophy

**Modern CSS is powerful enough.** No build step required. That's the bet — and three products in, it's paying off.

1. **Zero build tools.** No Sass, no Less, no PostCSS, no Tailwind. Write CSS the browser already understands.
2. **Semantic over utility-first.** A class says what an element *is* (`btn btn--negative`), not how it looks (`px-4 bg-red-500`). HTML stays readable.
3. **Tokens for everything.** Color, spacing, type, z-index, easings — all custom properties. Change one variable, the cascade does the rest.
4. **Cascade layers, not specificity wars.** `@layer reset, base, components, utilities;` — declared order sets the winner. No `!important`.
5. **OKLCH for color.** Perceptual lightness, chroma, hue. Define raw `--lch-*` triplets, build semantic colors on top with `oklch()`.
6. **Dark mode by flipping values.** Don't re-skin components. Re-declare the `--lch-*` triplets inside `@media (prefers-color-scheme: dark)` (or under `[data-theme="dark"]`) and the whole system shifts.
7. **Logical properties always.** `padding-inline`, `margin-block`, `inset-inline-start` — writing-mode aware, future-proof.
8. **Modern CSS replaces JS.** `:has()`, `@starting-style`, container queries, native nesting, `color-mix()`, masks — let the platform do the work.

## What This Plugin Deliberately Avoids

- **Sass / Less / Stylus** — native nesting, custom properties, and `calc()` cover it.
- **PostCSS / autoprefixer / any build step** — target evergreen browsers, ship the CSS as-is.
- **Tailwind / utility-first as the foundation** — utilities exist as *exceptions* (`.flex`, `.hide`), never as the design system.
- **Hardcoded colors** — no raw hex/rgb in components. Everything routes through `--lch-*` + semantic tokens.
- **`!important`** — if you reach for it, the layer order is wrong. Fix the cascade instead.
- **Magic numbers** — no bare `z-index: 9999`, no one-off `padding: 13px`. Name it, scale it, token it.
- **Deep nesting / utility-soup** — keep selectors flat and intentional.

When generating CSS, default to these omissions. Don't offer them as alternatives.

## The 5-Skill Catalog

These skills compose. Start with `css-design-system` on a blank project, then deepen.

| Skill | One-liner | Triggers on |
|---|---|---|
| **`css-design-system`** | Blank-slate bootstrap — scaffold reset, base, tokens, utilities and the `@layer` architecture. *The flagship.* | "set up CSS", "design system from scratch", "scaffold stylesheets", "no Tailwind", "CSS architecture", new/empty project styling |
| **`css-design-tokens`** | OKLCH color system, `--lch-*` triplets + semantic names, spacing/type/z-index scales, dark mode, `color-mix()` palette derivation. | "design tokens", "color system", "OKLCH", "dark mode", "theming", "spacing/type scale", "semantic colors", `data-theme` |
| **`css-components`** | Semantic component classes driven by CSS variables, BEM-like variants that only override vars, container queries. | "build a button/card/dialog", "component CSS", "variants", "BEM", "reusable component class", "variable-driven styling" |
| **`css-modern-features`** | `:has()`, `@starting-style`, container queries, native nesting, `color-mix()`, masks, logical properties — modern CSS that replaces JS. | "`:has()`", "`@starting-style`", "container queries", "native nesting", "CSS masks", "replace JS with CSS", "logical properties" |
| **`css-guardrails`** | Anti-patterns, naming conventions, decision rules, a numbered review checklist, a11y guardrails. | "review my CSS", "is this good CSS", "why is this `!important`", "fix the cascade", "make this on-convention", lint/audit |

## Core Principles When You Write CSS

1. **One way to do things.** Don't present three approaches — pick the convention below and implement it. Color is OKLCH. Layout spacing is logical properties. Layering is `@layer`. Theming is flipping `--lch-*`.

2. **Raw triplets → semantic tokens → component variables.** Three tiers, always. `--lch-blue: 54% 0.15 255` → `--color-link: oklch(var(--lch-blue))` → `--btn-background: var(--color-link)`. Components never touch raw color.

3. **Variants override variables, nothing else.** A `.btn--negative` sets `--btn-background` and stops. It never redeclares padding, radius, or display. This is the mini-API.

4. **Flat files, one per concept.** `_reset.css`, `base.css`, `colors.css`, `utilities.css`, `buttons.css`, `inputs.css`. No subdirectories, no partials, no import trees. Named exactly what they do.

5. **Reproduce real values.** Use the actual OKLCH triplets and property names from the living proofs below — not invented approximations.

6. **Co-locate.** Dark mode, hover, responsive, and `:has()` rules live *with* the component they affect, inside its file via native nesting.

## Living Proof (provenance only)

The patterns here are distilled from three shipped 37signals products — cite them as the *origin* of a pattern, never as a framework requirement:

- **Campfire** — introduced OKLCH, custom properties, View Transitions.
- **Writebook** — added container queries and `@starting-style`.
- **Fizzy** — introduced cascade layers, `color-mix()`, and complex `:has()`.

Together: **~14,000 lines of CSS across 105 files, zero build tools.** Each release adopts more modern CSS while keeping the same no-build philosophy. When you reference one, say *"From Fizzy's buttons:"* — describe only the CSS contract (e.g. *"when `html` gets `data-theme="dark"`"*), never how any framework toggled it.
