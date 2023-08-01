<script>
  export let value
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte"
 
  let anchor
</script>

{#if value}
  <div bind:this={anchor} class="wrapper">
    {#each value as file}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <div class="attachment"> {file.extension.toUpperCase()} </div>
    {/each}
  </div>
{/if}

<style>
  .wrapper {
    flex: auto;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    white-space: nowrap;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }

  .attachment {
    display: flex;
    align-items: center;
    border: 1px solid darkcyan;
    border-radius: 4px;
    height: 50%;
    padding: 0 0.5rem 0 0.5rem;
  }
  .attachment:hover {
    border: 1px solid lime;
  }
</style>