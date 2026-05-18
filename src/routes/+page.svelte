<script lang="ts">
  import { onMount } from 'svelte';
  import { gsap } from '$lib/gsap';

  let container: HTMLElement;
  let cursor: HTMLElement;
  const mm = gsap.matchMedia();

  onMount(() => {
    const ctx = gsap.context(() => {

      mm.add({ reduceMotion: '(prefers-reduced-motion: reduce)' }, (context) => {
        if (context.conditions?.reduceMotion) return;

        gsap.from('.grid > *', {
          autoAlpha: 0,
          y: 10,
          duration: 0.5,
          stagger: { each: 0.06, from: 'start' },
          ease: 'power2.out',
        });
      });

      const xTo = gsap.quickTo(cursor, 'x', { duration: 0.6, ease: 'power3' });
      const yTo = gsap.quickTo(cursor, 'y', { duration: 0.6, ease: 'power3' });

      window.addEventListener('mousemove', (e) => {
        xTo(e.clientX);
        yTo(e.clientY);
      });

    }, container);

    return () => {
      ctx.revert();
      mm.revert();
    };
  });
</script>

<svelte:head>
  <title>grid-grid</title>
  <meta name="description" content="A baseline-locked layout system for SvelteKit." />
</svelte:head>

<div class="cursor" bind:this={cursor}></div>

<div class="page" bind:this={container}>

  <div class="grid">
    <div class="name">
      <span class="t-3xl">grid—grid</span>
    </div>
    <div class="desc">
      <span class="t-xl muted">Baseline-locked layout system</span>
    </div>
    <div class="stack">
      <span class="t-xl muted">SvelteKit · GSAP</span>
    </div>
    <div class="year">
      <span class="t-xl muted">{new Date().getFullYear()}</span>
    </div>
  </div>

  <div class="grid">
    <div class="hero">
      <span class="t-4xl">BASE<br>LINE</span>
    </div>
    <div class="sub">
      <span class="t-xl muted">8px grid ·<br>12 columns</span>
    </div>
  </div>

</div>

<style>
  .name  { grid-column: span 3; }
  .desc  { grid-column: span 5; }
  .stack { grid-column: span 2; }
  .year  { grid-column: span 2; }

  .hero  { grid-column: span 9; grid-row: span 2; align-self: flex-end; }
  .sub   { grid-column: span 3; grid-row: span 2; }

  .cursor {
    position: fixed;
    top: 0;
    left: 0;
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: var(--color-fg);
    pointer-events: none;
    z-index: 9999;
    translate: -50% -50%;
  }

  .muted {
    color: var(--color-fg-muted);
  }
</style>
