<script>
  import { createEventDispatcher } from 'svelte'

  export let value
  export let inEdit
  export let formattedValue
  export let width 
  export let padded = false
  export let placeholder = "Enter ... "

  const dispatch = createEventDispatcher()

  const sendChanges = ( e ) => {
    dispatch( "change", { value: value } )
  }

</script>

<div class="control" class:inEdit style:width >
  {#if inEdit }
    <input on:keypress={sendChanges} class="inline-edit" class:padded {placeholder} bind:value />
  {:else}
    <p class="value"> {formattedValue || value || "" } </p>
  {/if}
</div>

<style>
  .control {
    flex: auto;
    display: flex;
    align-items: center;
    justify-content: var(--super-column-alignment);
    transition: all 130ms;
    overflow: hidden;
    box-sizing: border-box;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    min-width: 0;
  }
  .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }

  .value {
    box-sizing: border-box;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  .inline-edit {
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