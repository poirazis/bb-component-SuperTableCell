<script>
  import { createEventDispatcher, onMount } from 'svelte'

  export let value
  export let cellState

  export let inEdit
  export let formattedValue

  const dispatch = createEventDispatcher()

  let original

  onMount(() => {
    original = value
  })

  function submit(event) {
    event.preventDefault()
		if (value != original) {
			dispatch('change', { value: value })
		} else {
      dispatch("cancelEdit")
    }
  }

  function keydown(event) {
    if (event.key == 'Escape') {
      event.preventDefault()
      value = original
      dispatch("cancelEdit")
    } else if ( event.key == "Enter" ) {
      event.preventDefault()
      dispatch('change', { value: value })
    }
  }
	
	function focus(element) {
		element.focus()
	}
</script>

<div class="control">
  {#if inEdit }
    <input class="inline-edit" on:keydown={keydown} bind:value on:blur={submit} use:focus/>
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
    width: 100%;
    display: flex;
    align-items:  center;
    justify-content: var(--super-column-alignment);
    white-space: nowrap;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
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
    font: inherit;
    cursor: pointer;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>