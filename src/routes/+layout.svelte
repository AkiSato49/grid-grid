<script lang="ts">
  import '../app.css';
  import { onMount } from 'svelte';
  import { onNavigate, afterNavigate } from '$app/navigation';
  import { gsap } from '$lib/gsap';
  import { theme } from '$lib/theme';
  import { devMode } from '$lib/dev';

  let { children } = $props();

  const mm = gsap.matchMedia();

  onMount(() => {
    document.documentElement.dataset.theme = theme;

    if (devMode) {
      document.documentElement.dataset.dev = 'true';

      document.querySelectorAll('.grid').forEach((grid) => {
        const overlay = document.createElement('div');
        overlay.className = 'dev-grid-overlay';
        overlay.setAttribute('aria-hidden', 'true');
        for (let i = 1; i <= 12; i++) {
          const col = document.createElement('div');
          col.textContent = String(i);
          overlay.appendChild(col);
        }
        grid.appendChild(overlay);
      });
    }

    mm.add(
      { reduceMotion: '(prefers-reduced-motion: reduce)' },
      (ctx) => {
        if (ctx.conditions?.reduceMotion) gsap.globalTimeline.timeScale(0);
        else gsap.globalTimeline.timeScale(1);
      }
    );
  });

  $effect(() => {
    return () => mm.revert();
  });

  // Exit: fires after data loads, before DOM swap.
  // Returns a Promise — SvelteKit holds the DOM swap until it resolves.
  onNavigate(() => {
    if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

    return new Promise<void>((resolve) => {
      gsap.to('main', {
        autoAlpha: 0,
        y: -8,
        duration: 0.25,
        ease: 'power2.in',
        onComplete: resolve,
      });
    });
  });

  // Enter: fires after the DOM is updated with the new page.
  afterNavigate(() => {
    if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

    gsap.fromTo(
      'main',
      { autoAlpha: 0, y: 8 },
      { autoAlpha: 1, y: 0, duration: 0.35, ease: 'power2.out', clearProps: 'all' }
    );
  });
</script>

<a href="#main" class="skip-nav">skip to content</a>

<main id="main">
  {@render children()}
</main>

<style>
  .skip-nav {
    position: absolute;
    left: -9999px;
    top: var(--sp-2);
    z-index: 9999;
    padding: var(--sp-1) var(--sp-2);
    background: var(--color-bg);
    color: var(--color-fg);
    font-size: var(--t-sm);
  }

  .skip-nav:focus {
    left: var(--sp-2);
  }

  main {
    display: contents;
  }
</style>
