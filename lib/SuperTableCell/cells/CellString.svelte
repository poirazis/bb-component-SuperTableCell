<script>
  export let value = ""
  export let cellState
  export let formattedValue
  export let width 
  export let padded = false
  export let placeholder = "Enter ... "
  export let debounced

  const focus = (node) => {
    node.focus();
  }

  let timer;
	const debounce = e => {
    if (debounced) {
      clearTimeout(timer);
      timer = setTimeout(() => {
        value = e.target.value
      }, debounced ?? 0 );
    }
    else {
     value = e.target.value
    }
	}

</script>

<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  class="control" 
  class:inEdit={ $cellState == "Editing" }
  style:max-width={width} 
  tabIndex="0" 
  on:focus={ cellState.focus }
>

  {#if $cellState == "Editing" }
    <input class="inline-edit" {value} class:padded {placeholder} on:input={debounce} use:focus on:blur={cellState.unfocus} />
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
    overflow: hidden;
    box-sizing: border-box;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    min-width: 0;
    border: 1px solid transparent;
  }

  .value {
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
  .inEdit {
    color: var(--spectrum-global-color-gray-900);
    border-color: var(--spectrum-alias-border-color-mouse-focus);
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>