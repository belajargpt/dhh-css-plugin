---
name: css-design-tokens
description: "Use when defining or refactoring design tokens, color systems, theming, or dark mode in pure CSS — OKLCH colors, --lch-* triplets, oklch() with alpha, semantic color names (ink, canvas, link, negative, positive, selected, highlight), hue scales, spacing tokens (--inline-space, --block-space, ch/rem, logical properties), type scale, z-index scale, easings, shadows, focus rings, color-mix() palette derivation, light-dark(), and data-theme / prefers-color-scheme dark mode. Framework-agnostic 37signals-style vanilla CSS, no build tools."
---

# CSS Design Tokens

> Store the three color numbers, not the color. Name roles, not values. Flip one layer and the whole UI re-themes.

A design-token layer is a single `:root { … }` block of custom properties that every component reads from and nothing hardcodes around. Get this layer right and theming, dark mode, and palette tweaks become one-line edits instead of find-and-replace marathons. The patterns below are mined verbatim from 37signals' Campfire, Writebook, and Fizzy — pure CSS, zero build tooling, no Sass/PostCSS/Tailwind.

Load these tokens however your stack loads CSS — a bundler entry that imports each file, `<link>` tags in document order, or your framework's global stylesheet import. The only hard rule CSS itself enforces: define tokens early so everything downstream can reference them.

## The OKLCH approach: store the LCH triplet, wrap in oklch()

Every color is defined in **two layers**. First a bare **LCH triplet** — just three numbers, no function wrapper — stored in an `--lch-*` variable. Then a **named color** that wraps that triplet in `oklch(...)`.

```css
:root {
  --lch-black: 0% 0 0;                     /* the raw triplet: L C H */
  --color-black: oklch(var(--lch-black));  /* wrapped into a usable color */
}
```

The three channels of `L C H`:

| Channel | Range | Meaning |
| ------- | ----- | ------- |
| **L** Lightness | `0%`–`100%` | Perceptual lightness. `0%` black, `100%` white. This is the **scale axis** — moving a hue up/down the scale = changing L. |
| **C** Chroma | `0`–`~0.5` | Colorfulness. `0` is grey, higher is more saturated. Peaks around `0.2`–`0.26` for vivid mids; tints sit near `0`. |
| **H** Hue | `0`–`360` | The hue angle. Red ≈ 30, green ≈ 145, blue ≈ 255, purple ≈ 308. |

**Why store the triplet and not the finished color?** Two payoffs:

1. **Dark mode is a one-layer flip.** Re-point a single `--lch-*` value and *every* `oklch(var(--lch-…))` that references it updates automatically — no component rule changes.
2. **Alpha at the point of use.** Because the variable expands to three bare channels, you append `/ <alpha>` right where you use it:

```css
--color-link-50: oklch(var(--lch-blue) / 0.5);
--shadow: 0 0 0 1px oklch(var(--lch-black) / 5%);  /* same trick, percent alpha */
```

OKLCH keeps hue **perceptually consistent** when you adjust lightness — a tint and its full-strength sibling stay the same color, no color-picker round-trips. It is also wide-gamut, reaching colors sRGB hex can't express.

## Semantic naming: roles on top of the raw scale

The raw `--lch-*` scale is the **primitive** layer. Components must never read it. On top sit two more layers: **named colors** (the scale, wrapped) and **semantic abstractions** (role names that point at a scale step).

```css
/* Named — the wrapped scale */
--color-ink: oklch(var(--lch-ink-darkest));   /* "ink" with no suffix = darkest = primary text */
--color-ink-lighter: oklch(var(--lch-ink-lighter));

/* Abstractions — role names pointing at a step */
--color-canvas:   oklch(var(--lch-canvas));
--color-link:     oklch(var(--lch-blue-dark));
--color-negative: oklch(var(--lch-red-dark));
--color-positive: oklch(var(--lch-green-dark));
--color-selected: oklch(var(--lch-blue-lighter));
--color-highlight: oklch(var(--lch-yellow-lighter));
```

The role names mined from Fizzy: **ink** (text), **canvas** (page background), **link**, **negative** (errors/destructive), **positive** (success), **selected** (selection tint), **highlight** (marker/emphasis), plus `marker`, `considering`, `golden`. A component says `color: var(--color-ink)` and `background: var(--color-canvas)` — it states *what role the color plays*, never which hue it is. Retune `--lch-blue-dark` once and every link, focus ring, and selected state follows, because they all resolve through the semantic name. (Full semantic list: `references/color-system.md`.)

```
--lch-blue-dark  →  --color-link  →  component: color: var(--color-link)
   primitive          semantic              consumer (only this layer in components)
```

## The hue scale: one family at seven lightness steps

A hue family is the **same hue at seven lightness steps**, named for ink-on-canvas intuition: `darkest → darker → dark → medium → light → lighter → lightest`. Light mode climbs lightness `26% → 40% → ~56% → ~66% → ~84% → 92% → 96%`. `darkest` is for text; `lightest` is a faint background tint.

```css
--lch-blue-darkest: 26% 0.126 264;
--lch-blue-darker:  40% 0.166 262;
--lch-blue-dark:    57.02% 0.1895 260.46;
--lch-blue-medium:  66% 0.196 257.82;
--lch-blue-light:   84.04% 0.0719 255.29;
--lch-blue-lighter: 92% 0.026 254;
--lch-blue-lightest: 96% 0.016 252;
```

Note the structure: **lightness is the scale axis** (monotonic climb), **chroma peaks in the middle** (`medium` is most vivid; both ends taper so tints stay subtle and darks stay readable), and **hue drifts slightly** per step (a deliberate, hand-tuned warmth/coolness shift). Families present: **ink** (a near-neutral cool grey — tiny chroma on a blue hue), **uncolor** (a warm neutral), **red, yellow, lime, green, aqua, blue, violet, purple, pink.** All families, all steps, light + dark: `references/color-system.md`.

## Spacing tokens: ch inline, rem block, half/double via calc()

Spacing splits along the writing axes. **Inline** (horizontal in LTR) is sized in `ch`; **block** (vertical) in `rem`. Each base has a `-half` and `-double` derived with `calc()`.

```css
/* Spacing */
--inline-space: 1ch;
--inline-space-half: calc(var(--inline-space) / 2);
--inline-space-double: calc(var(--inline-space) * 2);
--block-space: 1rem;
--block-space-half: calc(var(--block-space) / 2);
--block-space-double: calc(var(--block-space) * 2);
```

**Why `ch` inline?** `ch` is the width of the `0` glyph in the current font, so horizontal gutters track the **text measure** — padding next to text scales with the font and stays optically proportional to character width. **Why `rem` block?** Vertical rhythm stays anchored to the **root font size**, unaffected by local font-size changes, so vertical spacing is consistent everywhere. Deriving half/double from one base means you retune the whole scale by editing a single value.

Always consume these through **logical properties** so the layout flips correctly under RTL and vertical writing modes:

```css
.component {
  padding-inline: var(--inline-space);   /* not padding-left/right */
  margin-block: var(--block-space);      /* not margin-top/bottom */
}
```

Type is the heart of the page, so even breakpoints can key off characters instead of device widths: `@media (max-width: 100ch) { … }` adapts the moment the line gets too wide — on any device, at any font size.

## Typography scale + the mobile bump

A named scale from `xx-small` to `xx-large`, all in `rem` so the whole thing keys off the root font size. System-font stacks only — no web-font loading, no FOUT.

```css
/* Text */
--font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
--font-serif: ui-serif, serif;
--font-mono: ui-monospace, monospace;

--text-xx-small: 0.55rem;
--text-x-small: 0.75rem;
--text-small: 0.85rem;
--text-normal: 1rem;
--text-medium: 1.1rem;
--text-large: 1.5rem;
--text-x-large: 1.8rem;
--text-xx-large: 2.5rem;
```

**The mobile bump pattern.** On narrow viewports the *small end* of the scale grows for legibility while the large end is left alone. The override lives **inside** `:root` as a nested media query — native CSS, no preprocessor. The large steps are intentionally re-stated at the same value to document that the decision was deliberate, not an oversight:

```css
@media (max-width: 639px) {
  --text-xx-small: 0.65rem;   /* up from 0.55rem */
  --text-x-small: 0.85rem;    /* up from 0.75rem */
  --text-small: 0.95rem;      /* up from 0.85rem */
  --text-normal: 1.1rem;      /* up from 1rem    */
  --text-medium: 1.2rem;      /* up from 1.1rem  */
  --text-large: 1.5rem;       /* unchanged       */
  --text-x-large: 1.8rem;     /* unchanged       */
  --text-xx-large: 2.5rem;    /* unchanged       */
}
```

## Named z-index, easings, shadows, focus rings, component sizes

**Named z-index scale.** Every stacking layer in the UI has an *intention name*, not a magic number, so a conflict is resolved by reading the list — not by guessing `9999`.

```css
/* Z-index */
--z-events-column-header: 1;
--z-events-day-header: 3;
--z-popup: 10;
--z-nav: 30;
--z-flash: 40;
--z-tooltip: 50;
--z-bar: 60;
--z-tray: 61;
```

**Easings** (custom cubic-béziers, credited to easingwizard.com in the source):

```css
--ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
--ease-out-overshoot: cubic-bezier(0.25, 1.75, 0.5, 1);
--ease-out-overshoot-subtle: cubic-bezier(0.25, 1.25, 0.5, 1);
```

**Shadow** — a stacked shadow of four black OKLCH layers at 5% alpha, each at increasing blur/offset for soft, realistic depth (note the inline alpha trick from earlier):

```css
--shadow: 0 0 0 1px oklch(var(--lch-black) / 5%),
          0 0.2em 0.2em oklch(var(--lch-black) / 5%),
          0 0.4em 0.4em oklch(var(--lch-black) / 5%),
          0 0.8em 0.8em oklch(var(--lch-black) / 5%);
```

**Focus rings** — composed from sub-tokens so any one dimension retunes independently, and the color reuses the semantic `--color-link`:

```css
--focus-ring-color: var(--color-link);
--focus-ring-offset: 1px;
--focus-ring-size: 2px;
--focus-ring: var(--focus-ring-size) solid var(--focus-ring-color);
```

**Component-size & layout tokens** — including tokens that compose other tokens:

```css
--btn-size: 2.65em;
--footer-height: 2.65rem;
--tray-size: clamp(12rem, 25dvw, 24rem);
--border: 1px solid var(--color-ink-lighter);
--dialog-duration: 150ms;
--main-padding: clamp(var(--inline-space), 3vw, calc(var(--inline-space) * 3));
--main-width: 1400px;
```

## Dark mode: flip the LCH lightness, mirror under system preference

Dark mode touches **zero component rules**. Because every `--color-*` is `oklch(var(--lch-…))`, re-pointing the `--lch-*` triplets cascades through everything. There are **two trigger paths with identical bodies**:

```css
/* 1. Explicit choice — beats the OS setting */
html[data-theme="dark"] {
  /* ...flipped --lch-* triplets... */
}

/* 2. System fallback — only when NO explicit choice was made */
@media (prefers-color-scheme: dark) {
  html:not([data-theme]) {
    /* ...the SAME flipped triplets, repeated... */
  }
}
```

The `:not([data-theme])` guard is the crux: the media query applies **only when no explicit theme is set**, so a user who chose light is never overridden by a dark OS setting, and vice-versa. The CSS contract is just one attribute — *when `<html>` has `data-theme="dark"`, the dark triplets apply; otherwise the OS preference governs.* Whatever sets that attribute (a persisted setting, an inline script, any toggle your stack provides) is outside the CSS.

**How the flip works:** the scale **inverts its lightness direction**. Light mode climbs `darkest 26% → lightest 96%`; dark mode climbs the *other way* — `darkest` becomes ~96% L (light text on a dark canvas) and `lightest` becomes ~25% L (a faint surface on dark). The **semantic names stay constant** — "darkest ink" is still your primary text color — only the raw lightness behind it is remapped.

**Before / after, ink:**

```css
/* LIGHT                              DARK (flipped) */
--lch-ink-darkest: 26% 0.05 264;      --lch-ink-darkest: 96.02% 0.0034 260;
--lch-ink-medium:  66% 0.008 258;     --lch-ink-medium:  62% 0.0122 260;
--lch-ink-lightest: 96% 0.002 252;    --lch-ink-lightest: 25% 0.0204 260;
```

**Before / after, blue** (drives `--color-link` via `--lch-blue-dark`):

```css
/* LIGHT                                DARK (flipped) */
--lch-blue-darkest: 26% 0.126 264;     --lch-blue-darkest: 95.93% 0.0217 252;
--lch-blue-dark:    57.02% 0.1895 260.46; --lch-blue-dark: 74% 0.1293 256;
--lch-blue-lightest: 96% 0.016 252;    --lch-blue-lightest: 25% 0.0318 264;
```

Canvas and inverted-ink swap too — note dark canvas is a deep blue-grey, not pure black:

```css
--lch-canvas: 20% 0.0195 232.58;
--lch-ink-inverted: var(--lch-black);
```

A handful of abstractions get explicit dark overrides on top of the flip (`--color-highlight`, `--color-golden`, terminal colors), and `--shadow` is re-declared with deeper spread and negative radii for dark surfaces. The full flipped scale for **every** family and all the dark re-points are in `references/color-system.md`.

## color-mix(): derive a whole palette from ONE token

When a component needs a coordinated background / text / border set, don't hand-pick three colors — mix them from a single seed. `color-mix()` blends two colors by percentage; pairing a seed hue with `--color-canvas`, `--color-ink`, or `transparent` derives a self-consistent set:

```css
.card {
  --card-color: oklch(var(--lch-blue-dark));
  --card-bg:     color-mix(in srgb, var(--card-color) 4%,  var(--color-canvas));
  --card-text:   color-mix(in srgb, var(--card-color) 30%, var(--color-ink));
  --card-border: color-mix(in srgb, var(--card-color) 33%, transparent);
}
```

Change `--card-color` to any hue and the background, text, and border all re-derive in harmony — and because the seeds (`--color-canvas`, `--color-ink`) themselves flip in dark mode, the derived palette adapts to dark mode for free. This is how a categorical card system stays tonally consistent across a dozen colors from one variable each.

## Modern alternative: light-dark()

For the simplest per-property cases, native `light-dark()` is even terser — it picks the first value in light mode, the second in dark, gated by the `color-scheme` property:

```css
:root { color-scheme: light dark; }

.panel {
  background: light-dark(oklch(var(--lch-white)), oklch(var(--lch-canvas)));
  color:      light-dark(var(--color-ink), var(--color-ink-inverted));
}
```

It's great for one-off pairs. For a *whole system*, the triplet-flip approach above still wins — it themes everything from one indirection layer instead of dualizing every property. Use `light-dark()` for spot fixes; use the LCH flip for the design system.

## Anti-Patterns

| Anti-pattern | Why it hurts | Do instead |
| ------------ | ------------ | ---------- |
| `color: #3b82f6` hardcoded in a component | Can't theme, can't dark-mode, can't retune | `color: var(--color-link)` |
| Same hex pasted in 20 files | Change one, miss nineteen | One `--lch-*` primitive, referenced everywhere |
| `oklch(54% 0.15 255)` literal in a component rule | Bypasses the indirection that makes dark mode free | Reference `--color-*`, never raw LCH, in components |
| `z-index: 9999` magic number | Stacking wars, no shared order | Add a named step to the `--z-*` scale |
| `padding-left` / `margin-top` | Breaks under RTL / vertical writing modes | `padding-inline` / `margin-block` |
| Duplicating a tint by eye | Drifts out of the family | Add a step to the hue scale, or `color-mix()` it |
| A component reaching past the semantic layer to a raw `--lch-*` | Couples the component to a hue; retuning the role breaks it | Components read **only** `--color-*` abstractions |

## When to use which color layer

- **Primitive `--lch-*`** — defined once in the token file. Edited only when retuning the palette or building the dark flip. **Never referenced by components.**
- **Named `--color-ink-dark` etc.** — the wrapped scale. Use when you genuinely need a *specific tonal step* (e.g. a border at `--color-ink-lighter`).
- **Semantic `--color-link`, `--color-negative` etc.** — the default for components. Reach for these first; they carry meaning and survive a palette retune.

## Related Skills

- **css-design-system** — the file organization and `@layer reset, base, components, modules, utilities` architecture these tokens live inside.
- **css-components** — how buttons, dialogs, and cards consume these tokens via local custom-property APIs.
- **css-modern-features** — `:has()`, `@starting-style`, container queries, `color-mix()`, masks, and the rest of the modern toolkit.
- **css-guardrails** — the rules that keep components from hardcoding values and reaching past the semantic layer.
