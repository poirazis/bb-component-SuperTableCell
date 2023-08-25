<script>
  import { createEventDispatcher } from 'svelte'

  export let value
  export let inEdit
  export let formattedValue

  const dispatch = createEventDispatcher()

  $: dispatch('change', { value: value} )

  function focus ( element ) {
    element.focus();
  }

</script>

<div class="control">
  {#if inEdit }
    <input class="inline-edit" placeholder="Search..." bind:value use:focus/>
  {:else}
    <div class="inline-value"> {formattedValue || value || "" } </div>
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
    flex: 1;
    display: flex;
    align-items:  center;
    justify-content: var(--super-column-alignment);
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  input.inline-edit {
    flex: auto;
    box-sizing: border-box;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    outline: none;
    background: none;
    color: inherit;
    border: none;

    cursor: pointer;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>