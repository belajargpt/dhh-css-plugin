# dhh-css

> **Write a real CSS design system the 37signals way — in any framework, with zero build tools.**

A framework-agnostic, pure-CSS design-system plugin for Claude Code. Five skills that make any LLM fluent in modern, semantic, no-build CSS — distilled from 37signals' Campfire, Writebook, and Fizzy.

---

## Philosophy

Modern CSS is powerful enough. No build step required.

While the industry sprints toward increasingly complex toolchains, this plugin walks calmly in the other direction — and uses what the browser already ships:

- **Zero build tools.** No Sass, no Less, no PostCSS, no Tailwind. Native CSS only.
- **Semantic over utility-first.** Classes say what an element *is* (`btn btn--negative`), not how it looks. HTML stays readable; changes cascade; variants compose.
- **Tokens for everything.** Color, spacing, type, z-index — all custom properties. Change one variable, the whole system updates.
- **OKLCH color + dark mode by flipping values.** Define raw `--lch-*` triplets, build semantic colors on top, then re-declare the triplets for dark mode. No per-component re-skinning.
- **Cascade layers, not specificity wars.** `@layer reset, base, components, utilities;` decides the winner by declared order — so `!important` never appears.
- **Logical properties everywhere.** `padding-inline`, `margin-block`, `inset-inline-start`.

It's **pure CSS** — drop it into plain static HTML, React, Vue, Svelte, or anything that loads a stylesheet. Load the files however your stack loads CSS: a bundler entry that imports each file, `<link>` tags, or your framework's global stylesheet import.

---

## Install

```bash
claude plugin marketplace add github:belajargpt/dhh-css-plugin
claude plugin install dhh-css
```

---

## Copy-ready starter kit

The plugin ships **15 real, framework-agnostic `.css` files** at [`dhh-css/assets/starter/`](dhh-css/assets/starter/) — not code blocks, actual files you copy into any project. Every value is real, mined verbatim from the 37signals stylesheets.

**Foundation** (load `_global.css` first): `_global.css` (the `@layer` line + the full OKLCH token system + dark mode) · `reset.css` · `base.css` · `layout.css` · `utilities.css`

**Reusable primitives** (one file, many uses): `buttons.css` · `inputs.css` · `icons.css` (a masked-icon engine + inline glyphs) · `animation.css` · `spinners.css` (CSS-mask loader, no JS) · `dialog.css` (`@starting-style`) · `avatars.css` · `bubble.css` · `dividers.css` · `tooltips.css`

The kit ships **primitives only** — domain-free building blocks. You build your *feature* components (a checkout panel, a kanban column) on top, in `@layer modules`, by setting primitive tokens rather than restyling them. See [`assets/starter/README.md`](dhh-css/assets/starter/README.md) for the load order and the primitives-vs-feature rule.

---

## The 5 Skills

Each skill auto-invokes based on trigger keywords in your prompts. Start with `css-design-system` on a blank project, then deepen with the rest.

### `css-design-system` — the flagship

Blank-slate bootstrap. Scaffolds a complete starting point on an empty project: a modern reset, sensible base styles, the design tokens, a thin utilities file, and the cascade-layer architecture that ties them together (`@layer reset, base, components, modules, utilities;`). Lays out flat, one-file-per-concept stylesheets — `_reset.css`, `base.css`, `colors.css`, `utilities.css`, `buttons.css` — with no subdirectories, no partials, no import trees. This is where every project begins.

**Triggers:** "set up CSS", "design system from scratch", "scaffold stylesheets", "CSS architecture", "no Tailwind", "style this blank project", "cascade layers setup".

### `css-design-tokens`

The OKLCH color system and every other scale. Defines raw `--lch-*` triplets (lightness, chroma, hue) and builds semantic colors on top — `--color-ink`, `--color-canvas`, `--color-link`, `--color-negative`, `--color-positive`, `--color-selected`. Adds alpha variants with `oklch(var(--lch-blue) / 0.5)`, derives whole palettes from one variable via `color-mix()`, and ships spacing (`--inline-space: 1ch`, `--block-space: 1rem`), type, and z-index scales. Dark mode is one block that flips the `--lch-*` triplets — under `@media (prefers-color-scheme: dark)` or a `[data-theme="dark"]` attribute.

**Triggers:** "design tokens", "color system", "OKLCH", "dark mode", "theming", "spacing scale", "type scale", "semantic colors", "`color-mix`", "`data-theme`".

### `css-components`

Semantic component classes driven entirely by CSS variables. A `.btn` declares its own mini-API of `--btn-*` variables with inline fallbacks (`padding: var(--btn-padding, 0.5em 1.1em)`); variants like `.btn--negative` or `.btn--reversed` *only override variables* and nothing else, so it's always clear what a variant changed. BEM-like naming (`block`, `block--modifier`, `block__element`), native nesting for co-located states, and container queries so components respond to their context rather than the viewport.

**Triggers:** "build a button/card/dialog", "component CSS", "variants", "BEM", "reusable component class", "variable-driven styling", "container query component".

### `css-modern-features`

The modern native CSS that replaces JavaScript and preprocessors. `:has()` to query an element about what's inside it (adapt a button when it contains an image, dim a row, lay out avatar groups by count). `@starting-style` + `transition ... allow-discrete` for enter/exit animations on `<dialog>` and `::backdrop`. Container queries, native nesting with `&`, `color-mix()`, CSS masks (a loading spinner with no JS, SVG, or GIF), `mix-blend-mode`, logical properties, capability queries (`any-hover`, `pointer: fine`), and character-based breakpoints (`@media (min-width: 100ch)`).

**Triggers:** "`:has()`", "`@starting-style`", "container queries", "native nesting", "CSS masks", "replace JS with CSS", "logical properties", "dialog animation".

### `css-guardrails`

The reviewer and the rulebook. Catches anti-patterns — `!important` wars, hardcoded hex/rgb in components, magic z-index numbers, deep nesting, utility-soup-as-foundation, fixed `px`, one-off variants — and replaces each with the on-convention fix: `@layer`, OKLCH semantic tokens, a named z-index scale, logical properties, component variables, and `ch`/`clamp`/container-query responsiveness. Ships a numbered CSS review checklist, naming conventions, decision rules, and accessibility guardrails (`:focus-visible`, `prefers-reduced-motion`, color contrast, screen-reader-only text).

**Triggers:** "review my CSS", "is this good CSS", "why is this `!important`", "fix the cascade", "make this on-convention", "audit/lint my stylesheet", "a11y check".

---

## Credits & References

This plugin stands on the shoulders of:

- **DHH & the 37signals team** — for shipping **Campfire** (chat), **Writebook** (publishing), and **Fizzy** (kanban) as living proof that modern CSS needs no build step. ~14,000 lines of CSS across 105 files, **zero build tools**.
- **Jason Zimdars** — for ["Modern CSS Patterns and Techniques in Campfire"](https://dev.37signals.com), the original walkthrough of OKLCH, `:has()`, native nesting, and capability queries in production.
- **["Vanilla CSS is All You Need"](https://zolkos.com)** — for tracing the evolution across all three products: Campfire's OKLCH and View Transitions, Writebook's container queries and `@starting-style`, Fizzy's cascade layers, `color-mix()`, and complex `:has()`.

Patterns are distilled and re-expressed as framework-agnostic Claude Code skills. The 37signals products are cited as the **provenance** of each pattern — the CSS itself is universal.

---

## License

MIT — plugin code is yours to use, modify, share.

Reference content adapted from the sources above (please respect their terms).
