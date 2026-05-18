<svelte:head>
  <title>grid-grid</title>
  <meta name="description" content="A baseline-locked layout system for SvelteKit." />
</svelte:head>

<div class="page">

  <div class="grid">
    <div class="name">
      <span class="t-3xl">grid—grid</span>
    </div>
    <div class="desc">
      <span class="t-xl muted">Baseline-locked layout system</span>
    </div>
    <div class="stack">
      <span class="t-xl muted">SvelteKit</span>
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

  .muted {
    color: var(--color-fg-muted);
  }
</style>
