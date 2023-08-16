<script>
  import { createEventDispatcher, onMount } from 'svelte'

  export let value
  export let inEdit
  export let formattedValue
  export let fieldSchema

  const dispatch = createEventDispatcher()
  let editing = false, original

  onMount(() => {
    original = value
  })

</script>

<div class="control">
  {#if inEdit}
    <div class="inline-value">
      {#if value}
        {#each value as row }
          <div class="item"> {row.primaryDisplay}</div>
        {/each}
      {/if}
    </div>
  {:else}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="inline-value">
      {#if value}
        {#each value as row }
          <div class="item"> {row.primaryDisplay}</div>
        {/each}
      {/if}
    </div>
  {/if}
</div>

<style>
  .control {
    flex: auto;
    display: flex;
    align-items: stretch;
    justify-content: stretch;
  }
  .inline-value { 
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    column-gap: 0.5rem;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }

  .item {
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: #3E6990;
    color: white;
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    max-height: 1.85rem;
    white-space: nowrap;
  }

  input.inline-edit {
    box-sizing: border-box;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    outline: none;
    background: none;
    color: inherit;
    border: none;
    font: inherit;
    cursor: pointer;
    height: 100%;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>