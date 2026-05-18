# grid-grid

SvelteKit starter template — baseline-locked grid system, typographic scale, theming, and GSAP page transitions. Use as a degit template: `npx degit your-username/grid-grid my-project`.

## Stack

- SvelteKit + Svelte 5 (runes) + TypeScript (strict)
- Plain CSS — no frameworks, design tokens via CSS custom properties (OKLCH)
- GSAP + ScrollTrigger for animation

## Commands

```bash
npm run dev      # dev server
npm run build    # production build
npm run preview  # preview build
```

## Project structure

```
src/
  app.css              # reset + imports (tokens → themes → typography → grid)
  template.svelte      # copy this for every new page
  lib/
    gsap.ts            # GSAP registered once — always import from here
    theme.ts           # one line to change default theme
  styles/
    tokens.css         # all CSS custom properties — only place raw values live
    themes.css         # all themes — [data-theme="dark|light"] blocks
    typography.css     # Satoshi font, text-box-trim, .t-xs → .t-4xl
    grid.css           # .page, .grid, .cell, .col-1…12, .row-1…5
  routes/
    +layout.svelte     # theme init, GSAP matchMedia, page transitions
    +error.svelte      # error page
    +page.svelte       # demo / scratch
```

## Conventions

- `.pi/conventions.json` enforced — run `/conventions audit` to check
- CSS tokens only — no hardcoded px/rem/color values in components
- Import GSAP from `$lib/gsap`, never from `gsap` directly
- All line-heights are multiples of `--bl` (8px)
- Svelte 5 runes only — no stores, no `export let`, no `$:`
- `import type` for type-only imports
- `satisfies` not `as` for type validation

## Do not

- Add `utils/`, `helpers/`, `common/`, `misc/`, `shared/` directories
- Hardcode colors, spacing, or font sizes — use tokens
- Import `gsap` directly — always use `$lib/gsap`
- Use `any` — use `unknown` and narrow
- Animate `top`/`left`/`width`/`height` — use transforms
- Use `opacity` for fade — use `autoAlpha`
- Leave `markers: true` in ScrollTrigger in production
