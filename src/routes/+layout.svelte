<script lang="ts">
  import '../app.css';
  import { onMount } from 'svelte';
  import { theme } from '$lib/theme';
  import { devMode } from '$lib/dev';

  let { children } = $props();

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
