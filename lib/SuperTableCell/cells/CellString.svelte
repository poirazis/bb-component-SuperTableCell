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
  <div class="inline-wrapper">
    <input class="inline-edit" on:keydown={keydown} bind:value on:blur={submit} use:focus/>
  </div>
{:else}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="inline-value" on:click={edit}>
    {value}
  </div>
{/if}

<style>
  .inline-wrapper {
    width: 100%;
  }

  .inline-value { 
    padding: 4px 0px;
    width: 100%;
  }

  .inline-edit {
    background-color: inherit;
    font-size: inherit;
    color: inherit;
    font-weight: inherit;
    text-align: inherit;
    box-shadow: none;
    box-sizing: border-box;
    width: 100%;
    border: var(--spectrum-textfield-border-size) solid;
    border-color: var(--spectrum-alias-border-color);
    border-radius: 4px;

  }

  .inline-edit:focus {
    padding: 4px;
    outline: none;
    border-width: 1px;
    border-style: solid;
    border-color: var(--spectrum-alias-border-color-mouse-focus);
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>