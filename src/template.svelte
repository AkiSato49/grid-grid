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
          // gsap.from('.grid > *', { autoAlpha: 0, y: 20, stagger: 0.08 });
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
    <!-- top — place children with grid-column / grid-row in <style> -->
  </div>

  <div class="grid">
    <!-- bottom -->
  </div>

</div>

<style>
  /* example placement — use semantic names, not col-* utilities
  .headline { grid-column: 1 / 9; grid-row: span 2; }
  .aside    { grid-column: 9 / 13; }
  */
</style>
