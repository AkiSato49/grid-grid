# grid-grid

A SvelteKit starter with a baseline-locked grid system, typographic scale, and theming. Use as a degit template.

## Variants

| branch | description |
|--------|-------------|
| `main` | base — grid, tokens, themes, typography |
| `gsap` | + GSAP, page transitions, cursor follower, entrance animations |

```bash
# base
npx degit AkiSato49/grid-grid my-project

# with GSAP
npx degit AkiSato49/grid-grid#gsap my-project

cd my-project && npm install
```

---

## How it works

### Baseline unit

Everything derives from one variable:

```css
--bl: 0.5rem; /* 8px */
```

All spacing, row heights, and line-heights are exact multiples of `--bl`. Text blocks occupy a predictable number of rows — no guessing, no tweaking.

---

### File structure

```
src/
  app.css                 ← reset + base body + style imports (in order)
  template.svelte         ← copy when making a new page
  lib/
    theme.ts              ← set default theme here (one line)
    dev.ts                ← toggle dev overlay (one line)
    gsap.ts               ← [gsap branch] GSAP registered once, import from here
  styles/
    tokens.css            ← all CSS custom properties
    themes.css            ← all themes ([data-theme="*"] blocks)
    typography.css        ← font, body defaults, .t-* scale classes
    grid.css              ← .page, .grid primitives
    dev.css               ← column/row overlay (toggled via data-dev attr)
  routes/
    +layout.svelte        ← global setup: theme, dev overlay, page transitions
    +error.svelte         ← error page using the grid
    +page.svelte          ← demo page
```

---

### Tokens (`src/styles/tokens.css`)

All raw values live here — nowhere else.

```css
--bl: 0.5rem;          /* baseline unit — 8px */

--t-xs … --t-4xl       /* font sizes */
--lh-xs … --lh-4xl     /* line-heights — always N × --bl */

--sp-1 … --sp-16       /* spacing — always N × --bl */

--row-h                /* grid row height = 8 × --bl = 64px */
```

**Rule:** never write a raw px/rem value in a component. Use a token.

---

### Themes (`src/styles/themes.css`)

All themes in one file, each as a `[data-theme]` block:

```css
[data-theme="dark"]  { --color-bg: …; --color-fg: …; --color-fg-muted: …; --color-border: …; }
[data-theme="light"] { --color-bg: …; --color-fg: …; --color-fg-muted: …; --color-border: …; }
```

OS preference handled via `@media (prefers-color-scheme)` — no flash before JS runs.

**Add a theme:** add a new `[data-theme="name"] { … }` block.

**Set default theme:** one line in `src/lib/theme.ts`:

```ts
export const theme = 'dark' satisfies 'dark' | 'light';
```

`satisfies` gives a compile error if the name doesn't match a defined theme.

---

### Typography (`src/styles/typography.css`)

Eight type scale steps, each baseline-locked:

| class    | font-size | line-height | baselines |
|----------|-----------|-------------|-----------|
| `.t-xs`  | 0.625rem  | 1rem        | 2bl       |
| `.t-sm`  | 0.75rem   | 1rem        | 2bl       |
| `.t-md`  | 1rem      | 1.5rem      | 3bl       |
| `.t-lg`  | 1.25rem   | 2rem        | 4bl       |
| `.t-xl`  | 1.75rem   | 2.5rem      | 5bl       |
| `.t-2xl` | 2.5rem    | 3rem        | 6bl       |
| `.t-3xl` | 3.5rem    | 4rem        | 8bl       |
| `.t-4xl` | 5rem      | 5.5rem      | 11bl      |

`text-box-trim: trim-start` + `text-box-edge: cap alphabetic` is applied on `body` — cuts dead space above cap-height so text optically snaps to the baseline grid. Firefox fallback via `::before` negative margin.

---

### Grid (`src/styles/grid.css`)

Three primitives, nothing else:

```html
<div class="page">
  <div class="grid">
    <!-- top zone -->
  </div>
  <div class="grid">
    <!-- bottom zone -->
  </div>
</div>
```

**`.page`** — `display: flex; flex-direction: column; justify-content: space-between; height: 100dvh`. Two `.grid` children → top and bottom zones, no fixed row count.

**`.grid`** — 12 columns, `grid-auto-rows: var(--row-h)`. Rows auto-generate.

**`.grid > *`** — `align-items: flex-end`. Content sits at the bottom of its cell.

**Placement** — use scoped `<style>` with semantic class names per page. No utility span classes.

```svelte
<style>
  .headline { grid-column: 1 / 9; grid-row: span 2; }
  .aside    { grid-column: 9 / 13; }
</style>
```

---

### Page transitions *(gsap branch only)*

Handled globally in `+layout.svelte` — no per-page setup needed.

#### The sequence

```
user clicks link
  ↓
data for next page loads in background
  ↓
onNavigate() fires  ← exit animation starts here
  exit: main fades out, slides up 8px (250ms, power2.in)
  returns a Promise — SvelteKit waits for it to resolve
  ↓
Promise resolves → SvelteKit swaps the DOM
  ↓
afterNavigate() fires  ← enter animation starts here
  enter: main fades in, slides up from 8px below (350ms, power2.out)
```

#### Why `onNavigate` not `beforeNavigate`

`beforeNavigate` fires the moment a link is clicked — before data fetches. If the next page is slow, the exit animation finishes and there's a blank wait. `onNavigate` fires *after* data is loaded, right before the DOM swap, so exit plays when the next page is ready.

#### The Promise mechanism

```ts
onNavigate(() => {
  return new Promise<void>((resolve) => {
    gsap.to('main', {
      autoAlpha: 0,
      y: -8,
      duration: 0.25,
      onComplete: resolve,  // resolve unblocks SvelteKit
    });
  });
});
```

SvelteKit treats a returned Promise from `onNavigate` as a signal to hold the DOM swap until it resolves. Without this, the DOM swaps mid-animation.

#### Customising per page

Override the exit on a specific page by calling `onNavigate` locally — SvelteKit runs both. To replace rather than layer, track a flag:

```ts
// in +layout.svelte: expose a writable flag
export let skipLayoutTransition = $state(false);

// in +layout.svelte onNavigate:
onNavigate(() => {
  if (skipLayoutTransition) return; // skip this page
  return new Promise<void>((resolve) => { /* … */ });
});
```

Or use a route-level `+layout.svelte` nested inside a route group to override transitions for a section.

#### Reduced motion

Both hooks bail early if `prefers-reduced-motion: reduce` is set:

```ts
if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
```

The global `mm.add` in `onMount` also sets `gsap.globalTimeline.timeScale(0)` — this catches any animation that slips through without an explicit check.

---

### GSAP *(gsap branch only)*

Import from `$lib/gsap` — never directly from the `gsap` package:

```ts
import { gsap, ScrollTrigger } from '$lib/gsap';
```

This ensures `registerPlugin` and `gsap.defaults` run once.

Defaults set at app level:

```ts
gsap.defaults({ duration: 0.6, ease: 'power2.out' });
```

**Rules:**
- Animate `x`, `y`, `scale`, `rotation`, `autoAlpha` — never `top`, `left`, `width`, `height`
- `autoAlpha` over `opacity` — also toggles `visibility: hidden` at 0
- Always scope with `gsap.context(fn, container)` — prevents selector leakage across components
- Always clean up with `ctx.revert()` + `mm.revert()` on unmount

---

### Page template (`src/template.svelte`)

Copy to `src/routes/your-page/+page.svelte`. The gsap branch version includes the full GSAP context + matchMedia setup. The base version is a plain scaffold.

---

### TypeScript

`strict` + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes` enabled.

- `satisfies` for narrowing, not `as` for assertion
- `unknown` not `any`
- `import type` for type-only imports
- Discriminated unions for state, not optional fields

---

### Svelte 5

- `$state()` not `let`
- `$derived()` not `$:`
- `$effect()` for side effects
- `$props()` not `export let`
- Snippets + `{@render}` not slots
- Load functions in `+page.server.ts`, not `onMount` fetch
- Form actions for mutations, not fetch in handlers
