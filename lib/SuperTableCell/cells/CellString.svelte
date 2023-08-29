<script>
  import { createEventDispatcher } from 'svelte'

  export let value
  export let inEdit
  export let formattedValue
  export let padded = true 

  const dispatch = createEventDispatcher()

  $: dispatch('change', { value: value} )

  function focus ( element ) {
    element.focus();
  }

</script>

<div class="control" class:inEdit>
  {#if inEdit }
    <input class="inline-edit" class:padded placeholder="Search..." bind:value use:focus />
  {:else}
    <div class="value"> {formattedValue || value || "" } </div>
  {/if}
</div>

<style>
  .control {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: var(--super-column-alignment);;
    transition: all 130ms;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }

  .value {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }
  .inline-edit {
    flex: auto;
    box-sizing: border-box;
    outline: none;
    background: none;
    color: inherit;
    border: none;
    cursor: pointer;
    overflow: hidden;
  }

  .inline-edit:focus {
    min-width: unset;;
  }

  .inline-edit::placeholder {
    color: var(--primaryColor);
    font-style: italic;
  }

  .padded {
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }
</style>