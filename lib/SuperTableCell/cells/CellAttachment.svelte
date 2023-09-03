<script>

  import { onMount } from "svelte"

 
  export let value
  let lightbox
  const _id = "test_gallery"
</script>

{#if value}
  <div class="wrapper" id={_id}> 
    {#each value as file}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <div class="attachment" on:click={lightbox.init( file.url )} > 
        {file.extension.toUpperCase()}
      </div>
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

  .thumbnail {
    width: 100px;
    height: 32px;
    overflow: hidden;
  }
</style>