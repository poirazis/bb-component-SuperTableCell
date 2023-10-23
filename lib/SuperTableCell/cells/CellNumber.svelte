<script>
  import { createEventDispatcher } from 'svelte'

  export let value = null
  export let formattedValue
  export let cellState
  export let unstyled
  export let placeholder = "Enter..."
  export let debounced

  const dispatch = createEventDispatcher()

  $: inEdit = $cellState == "Editing"

  let timer;
	const debounce = e => {
    value = e.target.value

    if (debounced) {    
      clearTimeout(timer);
      timer = setTimeout(() => {
        dispatch("change", value )
      }, debounced ?? 0 );
    }
    else {
     dispatch("change", value )
    }
	}

  function focus(element) {
    element.focus()
  }
</script>

<div class="superCell"
  class:inEdit
  class:unstyled
>
  {#if inEdit}
    <input 
      class="inline-edit" 
      inputmode="numeric"
      {value} 
      {placeholder} 
      on:input={debounce}
      on:blur
      use:focus
    />
  {:else}
   <div class="value"> {formattedValue || value || "" } </div>
  {/if}
</div>

<style>
  .value {
    flex: auto;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    box-sizing: border-box;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
  }
  .inline-edit {
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    outline: none;
    background: none;
    color: inherit;
    border: none;
    cursor: pointer;
    overflow: hidden;
    min-width: unset;
    text-align: right;
  }

  .inline-edit:focus {
    min-width: unset;
  }

  .inline-edit::placeholder {
    color: var(--primaryColor);
    font-style: italic;
  }
</style>