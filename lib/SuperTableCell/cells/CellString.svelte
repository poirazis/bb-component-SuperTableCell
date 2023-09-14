<script>
  import { createEventDispatcher } from "svelte";

  export let value = ""
  export let cellState
  export let formattedValue
  export let width 
  export let placeholder = "Enter ... "
  export let debounced
  export let unstyled = false

  const focus = (node) => {
    node.focus();
  }

  const dispatch = createEventDispatcher();

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
</script>

<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  class="superCell"
  class:unstyled 
  class:inEdit={ $cellState == "Editing" }
  style:max-width={width} 
>
  {#if $cellState == "Editing" }
    <input 
      class="inline-edit" 
      {value} 
      {placeholder} 
      on:input={debounce}
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
  }

  .inline-edit:focus {
    min-width: unset;
  }

  .inline-edit::placeholder {
    color: var(--primaryColor);
    font-style: italic;
  }
</style>