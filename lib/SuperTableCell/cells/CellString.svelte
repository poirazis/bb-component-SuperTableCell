<script>
  import { createEventDispatcher, onMount } from 'svelte'

  export let value
  export let cellOptions

  const dispatch = createEventDispatcher()
  let editing = false, original

  onMount(() => {
    original = value
  })

  function edit() {
    if (cellOptions.editable) {
      editing = true
      dispatch("enterEdit")
    }
  }

  function submit(event) {
    event.preventDefault()
		if (value != original) {
			dispatch('submit', { newValue: value })
		} else {
      dispatch("cancelEdit")
    }
		
    editing = false
  }

  function keydown(event) {
    if (event.key == 'Escape') {
      event.preventDefault()
      value = original
      editing = false
      dispatch("cancelEdit")
    } else if ( event.key == "Enter" ) {
      event.preventDefault()
      editing = false
      dispatch('submit', { newValue: value })
    }
  }
	
	function focus(element) {
		element.focus()
	}
</script>

  {#if editing}
    <input 
    style:padding-left={cellOptions.paddingLeft}
    style:padding-right={cellOptions.paddingLeft}
    class="inline-edit" on:keydown={keydown} bind:value on:blur={submit} use:focus/>
  {:else}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div on:click={edit} class="buttonCell" 
    style:padding-left={cellOptions.paddingLeft}
    style:padding-right={cellOptions.paddingLeft}
    > {value} </div>
  {/if}

<style>

  .inline-value { 
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }

  .inline-edit {
    min-height: 2.5rem;
    background-color: inherit;
    font-size: inherit;
    color: inherit;
    font-weight: inherit;
    text-align: inherit;
    box-shadow: none;
    box-sizing: border-box;
    width: 100%;
    white-space: nowrap;
  }

  input.inline-edit {
    box-sizing: border-box;
    outline: none;
    background: none;
    color: inherit;
    border: none;
    padding: 0;
    font: inherit;
    cursor: pointer;
    height: 100%;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>