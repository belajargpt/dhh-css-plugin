---
name: css-guardrails
description: "Use when reviewing, refactoring, cleaning up, linting, or auditing CSS — or when asked 'is this good CSS', 'review my stylesheet', 'why is this !important', 'fix the cascade', or 'make this on-convention'. Catches anti-patterns: !important wars, hardcoded hex/rgb in components, magic z-index numbers, deep nesting, utility-soup, duplicated colors, fixed px, one-off variants, and px media queries. Replaces them with @layer, OKLCH semantic tokens, a named z-index scale, logical properties, component variables, and ch/clamp/container-query responsiveness. Ships a numbered CSS REVIEW CHECKLIST and accessibility guardrails (focus-visible, prefers-reduced-motion, color-contrast, screen-reader-only). Framework-agnostic 37signals-style vanilla CSS — no build tools, no Sass/PostCSS/Tailwind."
---

# CSS Guardrails

> If a review can't be settled by reading the cascade out loud, the cascade is wrong — fix the architecture, not the symptom.

This is the **guardrails / review pass**. Run it over any CSS you generated or hand-edited before you call it done. It encodes the 37signals house style (mined from Campfire, Writebook, and Fizzy — ~14,000 lines, zero build tools) into mechanical rules: every "ugly" line should resolve to a layer, a token, or a component variable. If you find yourself reaching for `!important`, a raw hex, or a magic number, that's the signal a guardrail was skipped upstream.

Everything here is **pure native CSS**. It reads identically in a single static `.html` file or any stack. Load the stylesheets however your environment loads CSS — a bundler entry that imports each file, `<link>` tags in document order, or a global stylesheet import. The only hard ordering rule CSS itself enforces: the `@layer` line is the **first** thing the browser sees.

---

## Anti-Patterns

Each row is a rejection in code review. Left is what gets flagged; right is the on-convention fix.

### 1. `!important` to win a fight → declare a `@layer` order

`!important` is an admission that two rules collide and specificity lost. Layers resolve that by **declaration order**, not by escalation. A later layer always beats an earlier one regardless of selector weight — so utilities beat components without a single `!important`.

```css
/* ❌ BAD — escalating war, unwinnable, unreadable cascade */
.hide { display: none !important; }
.btn  { background: blue !important; }
```
```css
/* ✅ GOOD — order is the priority. Utilities sit in the top layer. */
@layer reset, base, components, modules, utilities;

@layer components { .btn  { background-color: var(--color-link); } }
@layer utilities  { .hide { display: none; } }   /* wins, no !important */
```

### 2. Hardcoded colors in components → semantic tokens

A component should never name a raw color. It names a **role** (`link`, `negative`, `ink`, `canvas`) and the token resolves it — which is also what makes dark mode free.

```css
/* ❌ BAD — hex/rgb welded into the component; dark mode impossible */
.alert { color: #c0392b; background: #fff; border: 1px solid #eee; }
```
```css
/* ✅ GOOD — semantic tokens resolve through OKLCH triplets */
.alert {
  color: var(--color-negative);
  background-color: var(--color-canvas);
  border: var(--border);
}
```

### 3. Magic z-index numbers → a named scale

`z-index: 9999` tells you nothing and loses to `99999` tomorrow. Name every stacking layer once, in one place, and resolve conflicts by reading the list.

```css
/* ❌ BAD — magic numbers, arms race */
.tooltip { z-index: 9999; }
.modal   { z-index: 99999; }
```
```css
/* ✅ GOOD — intention-named scale (verbatim from Fizzy) */
:root {
  --z-popup: 10;
  --z-nav: 30;
  --z-flash: 40;
  --z-tooltip: 50;
  --z-bar: 60;
  --z-tray: 61;
}
.tooltip { z-index: var(--z-tooltip); }
.flash   { z-index: var(--z-flash); }
```

### 4. Deep nesting (>3 levels) → flatten with `&`, `:is()`, `:has()`

Native nesting is great until it's a pyramid. Past ~3 levels, specificity creeps and the selector stops being readable. Flatten by lifting state to the component root and using `:is()`/`:has()`/`:where()`.

```css
/* ❌ BAD — five levels deep, brittle, high specificity */
.card { .body { .list { .item { .label { color: red; } } } } }
```
```css
/* ✅ GOOD — shallow; state read at the root, layout declared once */
.card__label { color: var(--card-text-color); }
.card:has(.card__closed) { --card-color: var(--color-card-complete); }
```

### 5. Utility-soup as the foundation → utilities are the exception

Utilities are single-purpose **overrides** that live in the top layer. They patch the last 5%. They are not the architecture. The architecture is semantic components whose HTML says what a thing **is** (`class="btn btn--negative"`), not how it looks.

```css
/* ❌ BAD — the component IS a pile of utilities; HTML is unreadable */
<button class="inline-flex items-center gap-2 px-4 py-2 rounded-full
               border bg-white text-black hover:brightness-95">Save</button>
```
```css
/* ✅ GOOD — one semantic class; utilities stay rare and in their layer */
@layer components {
  .btn {
    display: inline-flex; align-items: center; gap: 0.5em;
    padding: var(--btn-padding, 0.5em 1.1em);
    border-radius: var(--btn-border-radius, 99rem);
    border: var(--btn-border-size, 1px) solid var(--btn-border-color, var(--color-ink-light));
    background-color: var(--btn-background, var(--color-canvas));
    color: var(--btn-color, var(--color-ink));
  }
}
@layer utilities { .flex { display: flex; } .hide { display: none; } }
```
```html
<button class="btn btn--negative">Delete</button>
```

### 6. Duplicated colors across files → centralize in tokens

The same color pasted in three files is three bugs waiting to drift. Every color is defined **once** as an `--lch-*` triplet, wrapped once into a `--color-*` role, and referenced everywhere after.

```css
/* ❌ BAD — same blue, four files, four chances to diverge */
/* nav.css */    a { color: oklch(57% 0.19 260); }
/* card.css */   .link { color: #1a6dff; }
/* footer.css */ .more { color: rgb(26,109,255); }
```
```css
/* ✅ GOOD — one source of truth (triplet → role → use) */
:root {
  --lch-blue-dark: 57.02% 0.1895 260.46;
  --color-link: oklch(var(--lch-blue-dark));
}
a, .link, .more { color: var(--color-link); }
```

### 7. Fixed `px` everywhere → `rem` / `ch` / `dvw` + logical properties

`px` ignores the user's font size and writing direction. Use `rem` for vertical rhythm (keys off root font), `ch` for inline gutters (tracks the text measure), `dvw`/`dvh` and `clamp()` for fluidity — all expressed with **logical** properties so RTL and vertical writing modes work for free.

```css
/* ❌ BAD — physical, rigid, ignores font size and direction */
.panel { padding: 16px 24px; margin-top: 32px; width: 320px; }
```
```css
/* ✅ GOOD — logical + relative + fluid (Fizzy spacing tokens) */
:root {
  --inline-space: 1ch;   /* horizontal: one character width */
  --block-space: 1rem;   /* vertical: one root em */
}
.panel {
  padding-block: var(--block-space);
  padding-inline: var(--inline-space-double);
  margin-block-start: var(--block-space-double);
  inline-size: clamp(12rem, 25dvw, 24rem);
}
```

### 8. Inventing a one-off variant → flip the component's variables

When a component already exposes `--btn-*` knobs, a new look is a **variable override**, not a new class that re-declares geometry. Variants set tokens only; layout is declared once on the base.

```css
/* ❌ BAD — re-states every property to change a color */
.btn-danger {
  display: inline-flex; align-items: center; gap: 0.5em;
  padding: 0.5em 1.1em; border-radius: 99rem;
  border: 1px solid #c0392b; background: #c0392b; color: #fff;
}
```
```css
/* ✅ GOOD — flip only the variables (verbatim Fizzy variant shape) */
.btn--negative {
  --btn-background: var(--color-negative);
  --btn-border-color: var(--color-negative);
  --btn-color: var(--color-ink-inverted);
  --focus-ring-color: var(--color-negative);
}
```

### 9. `px` media-query breakpoints → `ch` measure or container queries

Device-width breakpoints break the moment someone bumps their font size or rotates a foldable. Break on the **content measure** (`ch`) or, better, on the component's own container so it's reusable in any slot.

```css
/* ❌ BAD — magic device widths, font-size-blind */
@media (max-width: 768px) { .sidebar { display: none; } }
```
```css
/* ✅ GOOD — break on the text measure, or on the container */
@media (max-width: 100ch) { .sidebar { display: none; } }

.bubble { container-type: inline-size; }
.bubble__number { font-size: clamp(10px, 50cqi, var(--bubble-number-max)); }
```

---

## Naming Conventions

BEM-like, lowercase, hyphenated. The class name should tell a reader what the thing **is** and how a piece relates to the whole.

| Pattern | Meaning | Example |
| --- | --- | --- |
| `.component` | A reusable component (the base) | `.btn`, `.card`, `.input`, `.dialog` |
| `.component--variant` | A modifier — flips variables only, never re-declares layout | `.btn--negative`, `.card--notification`, `.input--select` |
| `.component__element` | A part owned by the component | `.card__header`, `.btn__group`, `.switch__btn` |
| `--token` | A custom property — global token or component-scoped knob | `--color-link`, `--block-space`, `--btn-padding` |
| `--lch-*` | A raw OKLCH triplet (`L C H`, no wrapper) — the single source for a color | `--lch-blue-dark: 57.02% 0.1895 260.46;` |
| `.util` | A single-purpose override living in the `utilities` layer | `.flex`, `.gap`, `.hide`, `.pad` |
| `.for-screen-reader` | Visually-hidden, still announced to assistive tech | `<span class="for-screen-reader">Close</span>` |

Rule of thumb: if a class needs to win the cascade, it's a **utility** (top layer). If it changes a look, it's a **`--variant`** flipping tokens. If it's a child part, it's a **`__element`**. Reach for a brand-new component class only when none of those fit.

---

## Decision Rules

### Component vs. utility
- **Component** when the thing recurs as a named concept with internal structure or multiple properties that move together (a button, a card, a dialog). It lives in `@layer components` and exposes `--component-*` variables.
- **Utility** when you need a single declaration to win as a rare exception (`display: none`, a one-off `gap`). It lives in `@layer utilities`, does exactly one thing, and is allowed to override components — because the layer order, not specificity, lets it.
- Smell test: if you're writing more than one or two utilities to assemble a recurring UI block, that block wants to be a **component**.

### Token vs. one-off value
- **Token** when the value carries **meaning** (a role color, a spacing step, a z-layer, a timing) or appears more than once. Define it once in `:root` (colors as `--lch-*` triplet → `--color-*` role).
- **One-off literal** only when it's genuinely local and meaningless elsewhere — e.g. a `0.2em` chamfer inside a single component. Even then, prefer a component-scoped variable if a variant might want to retune it.
- Smell test: would changing this value in dark mode, RTL, or a theme require finding every copy? Then it's a **token**.

### Component variable vs. new class
- **Component variable** (`--btn-background`, `--card-color`) when an instance needs a different look but the **same geometry**. Override the variable on the instance or in a `--variant`; the base rule is untouched.
- **New class** only when the structure itself differs (a new `__element`, or a genuinely new component). Never clone a component just to change a color — that's the one-off-variant anti-pattern (#8).
- Smell test: if your "new" class would copy `display`, `padding`, and `border-radius` from an existing component, you wanted a **variable override**.

### Primitive vs. feature component
- **Primitive** when the block is domain-free and reused across screens (`.btn`, `.card`, `.icon`). It lives in `@layer components`, is named for what it *is*, and is parametrized by `--component-*` tokens. One file, many uses — safe to ship in a shared starter kit.
- **Feature component** when it composes primitives for one specific screen and names an app concept (`.checkout`, `.card-column`, `.comment`). It lives in `@layer modules` and **sets primitive tokens** (`.checkout .btn { --btn-background: … }`) instead of restyling them — keep it out of the shared layer.
- Smell test: does the selector name a thing in *your product's* domain? Then it's a feature component — don't let it leak into a primitive or a copied starter file.

---

## CSS Review Checklist

Run this top-to-bottom over any stylesheet you generated or edited. Each item is pass/fail — a fail is a fix, not a discussion.

1. **Layers present?** Is there a single `@layer reset, base, components, modules, utilities;` declared first, and is every rule opted into the right layer? Reject naked rules that rely on specificity to win.
2. **No `!important`?** Grep for it. Every hit must be justified by a third-party override you can't reach; otherwise it's a missing layer (anti-pattern #1).
3. **Tokens used, not literals?** No raw hex/rgb/hsl inside components. Colors resolve through `--color-*`; spacing through `--block-space`/`--inline-space`; z-index through the named scale (#2, #3, #6).
4. **OKLCH for color?** New colors defined as an `--lch-*` triplet wrapped in `oklch(var(--lch-*))`, so dark mode and alpha (`oklch(var(--lch-black) / 5%)`) come for free. No new sRGB hex.
5. **Logical properties?** `padding-inline` / `margin-block` / `inset-inline-start` / `block-size` / `inline-size` instead of `left`/`right`/`width`/`height`. No physical `padding: Tpx Rpx Bpx Lpx` shorthands in new code.
6. **Dark mode handled by tokens?** Color flips happen by re-pointing `--lch-*` triplets under `html[data-theme="dark"]` and the `@media (prefers-color-scheme: dark) html:not([data-theme])` fallback — **not** by overriding component rules.
7. **`a11y` focus-visible?** Every interactive element has a visible `:focus-visible` ring using `--focus-ring-*` tokens, with `outline-offset`. No `outline: none` left dangling without a replacement ring.
8. **Nesting depth OK?** No selector deeper than ~3 levels. Lift state to the component root with `:has()`/`:is()` instead of burrowing (#4).
9. **Responsive via `ch` / `clamp` / container?** Breakpoints use `ch` (the text measure) or container queries (`container-type: inline-size`, `cqi`), and fluid sizing uses `clamp()`/`min()`/`max()` — never magic `px` device widths (#7, #9).
10. **Variants flip variables?** Modifier classes set `--component-*` tokens only; they never re-declare `display`/`padding`/`border-radius`. Base geometry exists exactly once (#8).
11. **Motion respects users?** Transitions and animations are inside the component, and `@media (prefers-reduced-motion: reduce)` neutralizes them (see below).
12. **One source of truth per value?** No color, spacing, or timing duplicated across files. If it repeats, it's a token (#6).
13. **Primitives stay domain-free?** No shared primitive (a `.btn`/`.card`/`.icon` in `@layer components`) names an app concept or hardcodes a feature's color. Domain-specific blocks live in `@layer modules` and set primitive tokens instead.

If all thirteen pass, it's on-convention. If any fails, the fix is named in its anti-pattern, not improvised.

---

## Accessibility Guardrails

Accessibility is part of "good CSS," not a separate pass. These four are non-negotiable in review.

### Focus-visible rings (keyboard navigation)
Compose the ring from tokens so any dimension retunes once, and color it from the semantic role the control uses. Use `:focus-visible` (keyboard) rather than `:focus` (also mouse), and offset it so it's never clipped.

```css
:root {
  --focus-ring-color: var(--color-link);
  --focus-ring-offset: 1px;
  --focus-ring-size: 2px;
  --focus-ring: var(--focus-ring-size) solid var(--focus-ring-color);
}
.btn:has(input:focus-visible),
.btn:focus-visible {
  outline: var(--focus-ring-size) solid var(--focus-ring-color);
  outline-offset: var(--focus-ring-offset);
}
```

### prefers-reduced-motion
Honor the OS setting globally in the reset layer (verbatim from the Fizzy reset). Don't ship motion a vestibular-sensitive user can't turn off.

```css
@layer reset {
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
    html { scroll-behavior: initial; }
  }
}
```
(This is the one sanctioned `!important` zone — it exists to forcibly defeat any per-component motion regardless of layer.)

### Color contrast
The OKLCH system makes contrast auditable because **Lightness is an explicit axis**. Text uses the dark end of a family (`--color-ink` = `--lch-ink-darkest`) on the light canvas; in dark mode the triplet flips so `darkest` becomes ~96% L (light text) on a deep canvas. Pair text and background from opposite ends of the L scale, and verify foreground/background land at WCAG AA (4.5:1 body, 3:1 large). When mixing toward a tint, keep enough L separation:

```css
.card {
  --card-text-color: color-mix(in srgb, var(--card-color) 75%, var(--color-ink));
  color: var(--card-text-color);
  background-color: color-mix(in srgb, var(--card-color) 4%, var(--color-canvas));
}
```

### Screen-reader-only utility
Provide a visually-hidden-but-announced class for icon-only buttons and skip links. Components even key off it — an icon-only `.btn:has(.for-screen-reader)` becomes a circle automatically.

```css
@layer utilities {
  .for-screen-reader {
    position: absolute;
    inline-size: 1px;
    block-size: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
    border: 0;
  }
}
```
```html
<button class="btn btn--circle"><svg class="icon">…</svg><span class="for-screen-reader">Close</span></button>
```

---

## When to use

Reach for this skill to review or refactor an existing stylesheet, to answer "is this good CSS?", to clean up after a generation pass (run the **CSS Review Checklist** before shipping), to triage a cascade that "won't behave" (usually a missing `@layer` or an `!important` war), or to onboard hand-written CSS onto the convention without a rewrite. For building the system fresh rather than policing it, start with the sibling skills below.

---

## Related Skills

- **css-design-system** — the overall architecture: file organization, the five-layer `@layer` order, and how reset → base → components → modules → utilities fit together.
- **css-design-tokens** — defining the `--lch-*` triplets, `--color-*` roles, spacing, type scale, z-index scale, and the dark-mode flip these guardrails check against.
- **css-components** — authoring "variables all the way down" components and variants that flip tokens, so anti-patterns #5 and #8 never happen.
- **css-modern-features** — `:has()`, `@layer`, `color-mix()`, container queries, `@starting-style`, logical properties — the native tools the checklist requires.
