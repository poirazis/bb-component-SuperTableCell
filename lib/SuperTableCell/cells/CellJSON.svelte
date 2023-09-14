<script>
  import { createEventDispatcher } from "svelte";

  export let value = ""
  export let cellState
  export let formattedValue
  export let width 
  export let padded = false
  export let placeholder = "Enter ... "
  export let debounced
  export let unstyled = false

  const focus = (node) => {
    node.focus();
  }

  const dispatch = createEventDispatcher();

  let timer;
	const debounce = e => {
    if (debounced) {
      clearTimeout(timer);
      timer = setTimeout(() => {
        value = e.target.value
        dispatch("change", value )
      }, debounced ?? 0 );
    }
    else {
     value = e.target.value
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
    <input class="inline-edit" {value} {placeholder} on:input={debounce} use:focus on:blur={cellState.unfocus} />
  {:else}
    <div class="value"> JSON </div>
  {/if}
</div>

<style>

  .value {
    display: flex;
    align-items: center;
    box-sizing: border-box;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
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