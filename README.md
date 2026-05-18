# grid-grid

A SvelteKit starter with a baseline-locked grid system, typographic scale, theming, and GSAP — ready to use as a degit template.

```bash
npx degit your-username/grid-grid my-project
cd my-project && npm install
```

---

## How it works

### Baseline unit

Everything derives from one variable:

```css
--bl: 0.5rem; /* 8px */
```

All spacing, row heights, and line-heights are exact multiples of `--bl`. This means text blocks occupy a predictable number of rows in the grid — no guessing, no tweaking.

---

### File structure

```
src/
  app.css                   ← reset + base body + imports (orchestrator)
  template.svelte           ← copy this when making a new page
  lib/
    gsap.ts                 ← GSAP registered once, import from here
    theme.ts                ← change this one line to switch theme
  styles/
    tokens.css              ← all CSS custom properties
    themes.css              ← all themes in one file
    typography.css          ← font, body defaults, .t-* scale classes
    grid.css                ← .page, .grid, .grid > *, .col-*, .row-*
  routes/
    +layout.svelte          ← global setup: theme, GSAP, page transitions
    +error.svelte           ← error page using the grid system
    +page.svelte            ← demo page
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

All themes live in one file. Each is a `[data-theme]` block:

```css
[data-theme="dark"]  { --color-bg: …; --color-fg: …; --color-fg-muted: …; --color-border: …; }
[data-theme="light"] { --color-bg: …; --color-fg: …; --color-fg-muted: …; --color-border: …; }
```

OS default is handled via `@media (prefers-color-scheme)` so there's no flash before JS runs.

**To add a theme:** add a new `[data-theme="name"] { … }` block here.

**To set the default theme:** edit one line in `src/lib/theme.ts`:

```ts
export const theme = 'dark' satisfies 'dark' | 'light';
```

`satisfies` gives a compile error if the name doesn't exist as a theme.

---

### Typography (`src/styles/typography.css`)

Eight type scale steps, each with a baseline-locked line-height:

| class   | font-size  | line-height | baselines |
|---------|-----------|-------------|-----------|
| `.t-xs` | 0.625rem  | 1rem        | 2bl       |
| `.t-sm` | 0.75rem   | 1rem        | 2bl       |
| `.t-md` | 1rem      | 1.5rem      | 3bl       |
| `.t-lg` | 1.25rem   | 2rem        | 4bl       |
| `.t-xl` | 1.75rem   | 2.5rem      | 5bl       |
| `.t-2xl`| 2.5rem    | 3rem        | 6bl       |
| `.t-3xl`| 3.5rem    | 4rem        | 8bl       |
| `.t-4xl`| 5rem      | 5.5rem      | 11bl      |

`text-box-trim: trim-start` + `text-box-edge: cap alphabetic` is set on `body` — this cuts dead space above the cap-height so text optically snaps to the baseline grid.

---

### Grid (`src/styles/grid.css`)

Three primitives:

```html
<div class="page">       <!-- full-viewport flex column, space-between -->
  <div class="grid">     <!-- 12-col, auto rows at --row-h (64px) -->
    <div class="col-4 row-2"> … </div>
  </div>
  <div class="grid">     <!-- second grid sits at bottom of page -->
    …
  </div>
</div>
```

**`.page`** — `display: flex; flex-direction: column; justify-content: space-between; height: 100dvh`. Two `.grid` children → top zone and bottom zone. No fixed row count.

**`.grid`** — 12-column, `grid-auto-rows: var(--row-h)`. Rows generate automatically as content is added.

**`.grid > *`** — `align-items: flex-end; overflow: visible`. Content sits at the bottom of its cell.

**Column spans** — `.col-1` through `.col-12`

**Row spans** — `.row-1` through `.row-5` (multiples of 64px)

---

### GSAP (`src/lib/gsap.ts`)

Import GSAP from here — never directly from the `gsap` package:

```ts
import { gsap, ScrollTrigger } from '$lib/gsap';
```

This ensures `registerPlugin` and `gsap.defaults` run exactly once.

Defaults set at app level:

```ts
gsap.defaults({ duration: 0.6, ease: 'power2.out' });
```

**Rules:**
- Animate `x`, `y`, `scale`, `rotation`, `autoAlpha` — never `top`, `left`, `width`, `height`
- Use `autoAlpha` instead of `opacity` (also sets `visibility: hidden` at 0)
- Always scope animations to a container with `gsap.context(fn, container)`
- Always clean up with `ctx.revert()` on unmount

---

### Page template (`src/template.svelte`)

Copy this to `src/routes/your-page/+page.svelte` when starting a new page:

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { gsap, ScrollTrigger } from '$lib/gsap';

  // uncomment if this page loads data via +page.server.ts
  // let { data } = $props();

  let container: HTMLElement;
  const mm = gsap.matchMedia();

  onMount(() => {
    const ctx = gsap.context(() => {

      mm.add(
        {
          isDesktop:    '(min-width: 960px)',
          isMobile:     '(max-width: 959px)',
          reduceMotion: '(prefers-reduced-motion: reduce)',
        },
        (context) => {
          const { isDesktop, reduceMotion } = context.conditions!;
          if (reduceMotion) return;

          // animations here
        }
      );

    }, container);

    return () => {
      ctx.revert();
      mm.revert();
    };
  });
</script>

<svelte:head>
  <title>Page title</title>
  <meta name="description" content="Page description" />
</svelte:head>

<div class="page" bind:this={container}>
  <div class="grid">
    <!-- top -->
  </div>
  <div class="grid">
    <!-- bottom -->
  </div>
</div>
```

**Why `gsap.context(fn, container)`:** scopes all selector strings (`.grid > *`, `.t-4xl`, etc.) to the component root. Prevents animations leaking into other components.

**Why `mm.add` with `reduceMotion`:** respects `prefers-reduced-motion`. Users with vestibular disorders get no animation.

---

### Page transitions (`src/routes/+layout.svelte`)

Handled globally — no per-page setup needed.

```
click link
  → onNavigate fires (data already loaded, DOM not yet swapped)
  → exit animation plays (main fades + slides up 8px)
  → Promise resolves → SvelteKit swaps the DOM
  → afterNavigate fires
  → enter animation plays (main fades + slides up from 8px below)
```

Uses `onNavigate` (not `beforeNavigate`) because it fires after data loading — the exit animation doesn't start until the next page is ready to show. Returns a `Promise` that SvelteKit waits on before touching the DOM.

Both hooks bail early if `prefers-reduced-motion` is set.

---

### TypeScript conventions

- `strict: true` + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes` enabled
- Use `satisfies` for validation, not `as` for assertion
- Use `unknown` not `any`
- Use `import type` for type-only imports
- Use discriminated unions for state, not optional fields
- Use `as const` for literal config objects

---

### Svelte 5 conventions

- State: `$state()` not `let`
- Derived: `$derived()` not `$:`
- Side effects: `$effect()` not `$:`
- Props: `$props()` not `export let`
- Two-way binding: `$bindable()` on the prop
- Data fetching: `+page.server.ts` load functions, not `onMount` fetch
- Mutations: form actions, not fetch in event handlers
- Reusable template chunks: snippets + `{@render}`, not slots
