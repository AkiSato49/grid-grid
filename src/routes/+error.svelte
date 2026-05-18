<script lang="ts">
  import { page } from '$app/stores';
</script>

<div class="page">

  <div class="grid">
    <div class="status">
      <span class="t-sm">{$page.status}</span>
    </div>
  </div>

  <div class="grid">
    <div class="message">
      <span class="t-4xl">{$page.error?.message ?? 'something went wrong'}</span>
    </div>
  </div>

</div>

<style>
  .status  { grid-column: span 12; }
  .message { grid-column: span 12; grid-row: span 2; }
</style>
