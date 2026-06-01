# The Full OKLCH Color System — Reference

The complete color-token layer mined verbatim from a real 37signals Fizzy stylesheet. Pure CSS — OKLCH custom properties, no Sass/PostCSS/Tailwind, no build step. Every value here is copied from the source, not approximated. Use this as the canonical paste-in for a new project's color file, then retune the primitives.

The system has **three layers**, in this dependency order:

1. **Primitives** — bare `--lch-*` triplets (`L C H`, no function wrapper). The only layer that changes between light and dark.
2. **Named colors** — each primitive wrapped in `oklch(...)`. The full tonal scale, addressable.
3. **Semantic abstractions** — role names (`--color-link`, `--color-negative`, …) pointing at a step. **Components read only this layer.**

```
--lch-blue-dark   →   --color-link   →   component reads var(--color-link)
  (primitive)          (semantic)            (consumer)
```

---

## 1. Fixed colors — never change between themes

```css
/* OKLCH colors: Fixed */
--lch-black: 0% 0 0;
--lch-white: 100% 0 0;
```

---

## 2. Light-mode aliases

```css
/* OKLCH colors: Light mode */
--lch-canvas: var(--lch-white);
--lch-ink-inverted: var(--lch-white);
```

---

## 3. Light-mode hue scale — ALL families, all 7 steps (verbatim)

Each family runs `darkest → darker → dark → medium → light → lighter → lightest`. Lightness climbs monotonically (`26% → ~96%`); chroma peaks in the middle; hue drifts slightly per step. Families: **ink, uncolor, red, yellow, lime, green, aqua, blue, violet, purple, pink.**

```css
--lch-ink-darkest: 26% 0.05 264;
--lch-ink-darker: 40% 0.026 262;
--lch-ink-dark: 56% 0.014 260;
--lch-ink-medium: 66% 0.008 258;
--lch-ink-light: 84% 0.005 256;
--lch-ink-lighter: 92% 0.003 254;
--lch-ink-lightest: 96% 0.002 252;

--lch-uncolor-darkest: 26% 0.018 40;
--lch-uncolor-darker: 40.04% 0.0376 50.06;
--lch-uncolor-dark: 57.09% 0.0676 60.5;
--lch-uncolor-medium: 66% 0.0944 71.46;
--lch-uncolor-light: 83.97% 0.0457 80.84;
--lch-uncolor-lighter: 92% 0.014 90;
--lch-uncolor-lightest: 96% 0.012 100;

--lch-red-darkest: 26% 0.105 34;
--lch-red-darker: 40% 0.154 36;
--lch-red-dark: 59% 0.19 38;
--lch-red-medium: 66% 0.204 40;
--lch-red-light: 84.08% 0.0837 41.96;
--lch-red-lighter: 92% 0.03 44;
--lch-red-lightest: 96% 0.013 46;

--lch-yellow-darkest: 26% 0.0729 40;
--lch-yellow-darker: 40% 0.12 50;
--lch-yellow-dark: 58% 0.156 60;
--lch-yellow-medium: 74% 0.184 70;
--lch-yellow-light: 84% 0.12 80;
--lch-yellow-lighter: 92% 0.076 90;
--lch-yellow-lightest: 96% 0.034 100;

--lch-lime-darkest: 26% 0.064 109;
--lch-lime-darker: 40% 0.101 110;
--lch-lime-dark: 56.5% 0.142 111;
--lch-lime-medium: 68% 0.176 113.11;
--lch-lime-light: 83.92% 0.0927 113.6;
--lch-lime-lighter: 92% 0.046 114;
--lch-lime-lightest: 96% 0.034 115;

--lch-green-darkest: 26% 0.071 149;
--lch-green-darker: 40% 0.12 148;
--lch-green-dark: 55% 0.162 147;
--lch-green-medium: 66% 0.208 146;
--lch-green-light: 83.92% 0.0772 145.06;
--lch-green-lighter: 92% 0.044 144;
--lch-green-lightest: 96% 0.022 143;

--lch-aqua-darkest: 26% 0.059 214;
--lch-aqua-darker: 40% 0.093 212;
--lch-aqua-dark: 55.5% 0.122 210;
--lch-aqua-medium: 66% 0.152 208;
--lch-aqua-light: 83.88% 0.0555 206.02;
--lch-aqua-lighter: 92% 0.02 204;
--lch-aqua-lightest: 96% 0.012 202;

--lch-blue-darkest: 26% 0.126 264;
--lch-blue-darker: 40% 0.166 262;
--lch-blue-dark: 57.02% 0.1895 260.46;
--lch-blue-medium: 66% 0.196 257.82;
--lch-blue-light: 84.04% 0.0719 255.29;
--lch-blue-lighter: 92% 0.026 254;
--lch-blue-lightest: 96% 0.016 252;

--lch-violet-darkest: 26% 0.148 292;
--lch-violet-darker: 40% 0.2 290;
--lch-violet-dark: 58% 0.216 287.6;
--lch-violet-medium: 66% 0.206 285.52;
--lch-violet-light: 84.08% 0.0791 283.47;
--lch-violet-lighter: 92% 0.03 282;
--lch-violet-lightest: 96% 0.015 280;

--lch-purple-darkest: 26% 0.131 314;
--lch-purple-darker: 40% 0.178 312;
--lch-purple-dark: 58% 0.21 310;
--lch-purple-medium: 66% 0.258 308;
--lch-purple-light: 84.09% 0.0778 305.77;
--lch-purple-lighter: 92% 0.03 304;
--lch-purple-lightest: 96% 0.019 302;

--lch-pink-darkest: 26% 0.12 348;
--lch-pink-darker: 40% 0.16 346;
--lch-pink-dark: 59% 0.188 344;
--lch-pink-medium: 71.8% 0.2008 342;
--lch-pink-light: 84.04% 0.0737 340;
--lch-pink-lighter: 92% 0.03 338;
--lch-pink-lightest: 96% 0.02 336;
```

### Reading the scale

- **Lightness is the scale axis.** Per family the L% climbs `26 → 40 → ~56 → ~66 → ~84 → 92 → 96`. Names describe ink on canvas: `darkest` = text, `lightest` = faint background.
- **Chroma peaks in the middle.** Saturation maxes around `medium` (e.g. `--lch-purple-medium` chroma `0.258`) and tapers to both ends so the lightest tints stay subtle and the darkest stay readable.
- **`ink` is the near-neutral.** Tiny chroma (`0.05 → 0.002`) on a blue hue — a *cool grey*, so neutral chrome harmonizes with blue accents instead of looking dead grey.
- **`uncolor` is the warm neutral.** Warm hues (40–100) at low-ish chroma — a second neutral for sepia/warm surfaces.

---

## 4. Named colors — wrapping the triplets

The full scale, wrapped in `oklch()`. `--color-ink` with no suffix aliases `--color-ink-darkest`: "ink" means the darkest, most-readable text color. The same wrapping pattern applies to every family — `--color-blue-dark: oklch(var(--lch-blue-dark))` and so on for red, yellow, lime, green, aqua, violet, purple, pink, uncolor.

```css
/* Colors: Named */
--color-black: oklch(var(--lch-black));
--color-white: oklch(var(--lch-white));

--color-ink: oklch(var(--lch-ink-darkest));
--color-ink-darkest: oklch(var(--lch-ink-darkest));
--color-ink-darker: oklch(var(--lch-ink-darker));
--color-ink-dark: oklch(var(--lch-ink-dark));
--color-ink-medium: oklch(var(--lch-ink-medium));
--color-ink-light: oklch(var(--lch-ink-light));
--color-ink-lighter: oklch(var(--lch-ink-lighter));
--color-ink-lightest: oklch(var(--lch-ink-lightest));

--color-ink-inverted: oklch(var(--lch-ink-inverted));
```

---

## 5. Semantic abstractions — the layer components read

Role-named colors pointing at scale steps. The meaning (`link`, `negative`, `selected`) stays stable even if the underlying hue is retuned. **This is the full mined abstraction list.**

```css
/* Colors: Abstractions */
--color-canvas: oklch(var(--lch-canvas));
--color-negative: oklch(var(--lch-red-dark));
--color-positive: oklch(var(--lch-green-dark));
--color-link: oklch(var(--lch-blue-dark));
--color-selected-light: oklch(var(--lch-blue-lightest));
--color-selected: oklch(var(--lch-blue-lighter));
--color-selected-dark: oklch(var(--lch-blue-light));
--color-highlight: oklch(var(--lch-yellow-lighter));
--color-marker: oklch(var(--lch-red-medium));
--color-terminal-bg: oklch(98% 0.002 252);
--color-terminal-text: var(--color-ink);
--color-terminal-text-light: var(--color-ink-lighter);
--color-golden: oklch(89.1% 0.178 95.7);
--color-considering: oklch(var(--lch-blue-medium));
```

| Semantic token | Role | Resolves through |
| -------------- | ---- | ---------------- |
| `--color-canvas` | Page / surface background | `--lch-canvas` (= white, light) |
| `--color-ink` | Primary text | `--lch-ink-darkest` |
| `--color-link` | Links, focus ring color | `--lch-blue-dark` |
| `--color-negative` | Errors, destructive actions | `--lch-red-dark` |
| `--color-positive` | Success states | `--lch-green-dark` |
| `--color-selected` | Selection / checked tint | `--lch-blue-lighter` |
| `--color-selected-light` / `-dark` | Lighter / darker selection variants | `--lch-blue-lightest` / `--lch-blue-light` |
| `--color-highlight` | Emphasis / marker background | `--lch-yellow-lighter` |
| `--color-marker` | Marker pen | `--lch-red-medium` |
| `--color-considering` | "In consideration" state | `--lch-blue-medium` |
| `--color-golden` | Accent gold | literal OKLCH (re-pointed in dark) |
| `--color-terminal-*` | Terminal/code surfaces | literal + ink aliases |

---

## 6. Card colors — a categorical palette

Each card color maps to a different family's `medium` step (or a semantic alias), so labels are distinguishable but tonally consistent.

```css
/* Colors: Cards */
--color-card-default: oklch(var(--lch-blue-dark));
--color-card-complete: var(--color-ink-darker);
--color-card-1: oklch(var(--lch-ink-medium));
--color-card-2: oklch(var(--lch-uncolor-medium));
--color-card-3: oklch(var(--lch-yellow-medium));
--color-card-4: oklch(var(--lch-lime-medium));
--color-card-5: oklch(var(--lch-aqua-medium));
--color-card-6: oklch(var(--lch-violet-medium));
--color-card-7: oklch(var(--lch-purple-medium));
--color-card-8: oklch(var(--lch-pink-medium));
```

### Deriving a card's full palette from its one color — `color-mix()`

Each card needs only its single seed color; background, text, and border derive from it. Because the mix partners (`--color-canvas`, `--color-ink`) flip in dark mode, the derived palette adapts automatically.

```css
.card {
  --card-color: oklch(var(--lch-blue-dark));
  --card-bg:     color-mix(in srgb, var(--card-color) 4%,  var(--color-canvas));
  --card-text:   color-mix(in srgb, var(--card-color) 30%, var(--color-ink));
  --card-border: color-mix(in srgb, var(--card-color) 33%, transparent);
}
```

---

## 7. Highlighter palette — deliberately raw rgb()/rgba()

Paired foreground/background pen colors. These are intentionally fixed `rgb()`/`rgba()`, **not** OKLCH — pen ink should look identical in any theme, so it does not flip.

```css
/* Colors: Highlighter */
--highlight-1: rgb(136, 118, 38);
--highlight-2: rgb(185, 94, 6);
--highlight-3: rgb(207, 0, 0);
--highlight-4: rgb(216, 28, 170);
--highlight-5: rgb(144, 19, 254);
--highlight-6: rgb(5, 98, 185);
--highlight-7: rgb(17, 138, 15);
--highlight-8: rgb(148, 82, 22);
--highlight-9: rgb(102, 102, 102);

--highlight-bg-1: rgba(229, 223, 6, 0.3);
--highlight-bg-2: rgba(255, 185, 87, 0.3);
--highlight-bg-3: rgba(255, 118, 118, 0.3);
--highlight-bg-4: rgba(248, 137, 216, 0.3);
--highlight-bg-5: rgba(190, 165, 255, 0.3);
--highlight-bg-6: rgba(124, 192, 252, 0.3);
--highlight-bg-7: rgba(140, 255, 129, 0.3);
--highlight-bg-8: rgba(221, 170, 123, 0.3);
--highlight-bg-9: rgba(200, 200, 200, 0.3);
```

---

## 8. Syntax-highlighting tokens

Code-token colors mapped to semantic hues from the scale.

```css
/* Colors: Syntax highlighting */
--color-code-token__att: oklch(var(--lch-blue-dark));
--color-code-token__comment: oklch(var(--lch-ink-medium));
--color-code-token__function: oklch(var(--lch-purple-dark));
--color-code-token__operator: oklch(var(--lch-red-dark));
--color-code-token__property: oklch(var(--lch-purple-dark));
--color-code-token__punctuation: oklch(var(--lch-ink-dark));
--color-code-token__selector: oklch(var(--lch-green-dark));
--color-code-token__variable: oklch(var(--lch-red-dark));
```

---

## 9. Gradient tokens

```css
/* Colors: Generating gradient */
--color-gradient-1: oklch(var(--lch-violet-lighter));
--color-gradient-2: oklch(var(--lch-pink-lighter));
--color-gradient-3: oklch(var(--lch-purple-lighter));
--color-gradient-4: var(--color-canvas);
```

---

## 10. Dark mode — flip the primitives, nothing else

Dark mode re-points the `--lch-*` triplets only. Every `--color-*` is `oklch(var(--lch-…))`, so flipping the triplets cascades through the whole UI without touching one component rule. **Two trigger paths, identical bodies:**

```css
/* Explicit choice — overrides the OS setting */
html[data-theme="dark"] {
  /* ...flipped --lch-* triplets (below)... */
}

/* System fallback — only when no explicit choice was made */
@media (prefers-color-scheme: dark) {
  html:not([data-theme]) {
    /* ...the SAME flipped triplets, repeated verbatim... */
  }
}
```

The `:not([data-theme])` guard ensures the media query applies **only** when no explicit theme is set, so an explicit light choice is never overridden by a dark OS preference (and vice-versa). The CSS contract is one attribute on `<html>`; whatever sets it is outside CSS.

### How the flip works

The scale **inverts its lightness direction.** Light mode: `darkest 26% → lightest 96%`. Dark mode: the lightness runs the *other way* — `darkest` becomes ~96% (light text on dark canvas), `lightest` becomes ~25% (faint surface on dark). The **semantic names stay constant** — "darkest ink" is still primary text — only the raw lightness behind them is remapped. Canvas becomes a deep blue-grey, not pure black.

### Canvas / inverted-ink (dark)

```css
--lch-canvas: 20% 0.0195 232.58;   /* deep blue-grey, not #000 */
--lch-ink-inverted: var(--lch-black);
```

### Flipped families (verbatim)

**Ink** — `darkest` now ~96% L (light text), `lightest` now ~25% L (faint dark surface):

```css
--lch-ink-darkest: 96.02% 0.0034 260;
--lch-ink-darker: 86% 0.0061 260;
--lch-ink-dark: 73.97% 0.009 260;
--lch-ink-medium: 62% 0.0122 260;
--lch-ink-light: 40% 0.0148 260;
--lch-ink-lighter: 30% 0.0178 260;
--lch-ink-lightest: 25% 0.0204 260;
```

**Blue** — drives `--color-link` through `--lch-blue-dark`:

```css
--lch-blue-darkest: 95.93% 0.0217 252;
--lch-blue-darker: 86% 0.068 254;
--lch-blue-dark: 74% 0.1293 256;
--lch-blue-medium: 62% 0.159 258;
--lch-blue-light: 40% 0.094 260;
--lch-blue-lighter: 30% 0.0452 262;
--lch-blue-lightest: 25% 0.0318 264;
```

**Red** — drives `--color-negative` / `--color-marker`:

```css
--lch-red-darkest: 95.85% 0.0218 46;
--lch-red-darker: 86% 0.086 44;
--lch-red-dark: 73.95% 0.139 42;
--lch-red-medium: 62% 0.154 40;
--lch-red-light: 40% 0.088 38;
--lch-red-lighter: 30% 0.032 36;
--lch-red-lightest: 25% 0.011 34;
```

**Green** — drives `--color-positive`:

```css
--lch-green-darkest: 96.12% 0.035 143;
--lch-green-darker: 86% 0.082 144;
--lch-green-dark: 73.99% 0.117 145;
--lch-green-medium: 62% 0.1261 146;
--lch-green-light: 40% 0.065 147;
--lch-green-lighter: 30% 0.03 148;
--lch-green-lightest: 25% 0.018 149;
```

The complete dark set exists for **every** family — `uncolor`, `yellow`, `lime`, `aqua`, `violet`, `purple`, `pink` — following the same inversion (`darkest` → ~96% L, `lightest` → ~25% L, hue held near its light-mode value). The four above are representative; the file repeats the full block verbatim inside *both* the `html[data-theme="dark"]` selector and the `@media (prefers-color-scheme: dark) html:not([data-theme])` block. To build the rest, take each light-mode family and remap its lightness ramp to the dark ladder (`96 → 86 → 74 → 62 → 40 → 30 → 25`), keeping hue and lowering chroma toward the light end as the ink/blue/red/green examples show.

### Dark-mode semantic re-points

A few abstractions are overridden explicitly for dark, not just inherited through the triplet flip:

```css
--color-terminal-bg: var(--color-canvas);
--color-terminal-text-light: oklch(var(--lch-green-dark));
--color-golden: oklch(var(--lch-blue-medium));
--color-highlight: oklch(var(--lch-blue-lighter));
```

### Dark-mode shadow

Dark surfaces need a deeper, more spread stack with negative spread radii. The override re-declares `--shadow` entirely:

```css
--shadow: 0 0 0 1px oklch(var(--lch-black) / 0.42),
  0 0.2em 1.6em -0.8em oklch(var(--lch-black) / 0.6),
  0 0.4em 2.4em -1em oklch(var(--lch-black) / 0.7),
  0 0.4em 0.8em -1.2em oklch(var(--lch-black) / 0.8),
  0 0.8em 1.2em -1.6em oklch(var(--lch-black) / 0.9),
  0 1.2em 1.6em -2em oklch(var(--lch-black) / 1);
```

---

## 11. The contract, restated

- Components reference **semantic** `--color-*` names only. Never `--lch-*`, never literal `oklch()`, never hex.
- The `--lch-*` primitives are the **single source of truth** and the **only** thing dark mode edits.
- Alpha is spliced at the point of use: `oklch(var(--lch-black) / 5%)` — the variable expands to three bare channels, you append `/ <alpha>`.
- Fixed pen/highlighter colors stay raw `rgb()`/`rgba()` precisely *because* they must not flip.
- For one-off light/dark pairs, native `light-dark(lightValue, darkValue)` with `color-scheme: light dark` is a terser alternative — but the triplet flip themes the *whole* system from one indirection layer, which is why the design system uses it.
