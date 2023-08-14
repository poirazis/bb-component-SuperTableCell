<script>
  import { createEventDispatcher, onMount } from 'svelte'
  import Multiselect from "../../../node_modules/@budibase/bbui/src/Form/Core/Multiselect.svelte";
  import ActionButton from "../../../node_modules/@budibase/bbui/src/ActionButton/ActionButton.svelte"

  export let value
  export let options
  export let cell;

  const dispatch = createEventDispatcher()
  let original

  onMount(() => {
    original = value
  })


  function submit(event) {
    event.preventDefault()
		if (value != original) {
			dispatch('submit', { newValue: value })
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
      dispatch('submit', { newValue: value })
    }
  }

  function cancelEdit ( e ) {
    e.preventDefault(); 
    e.stopPropagation(); 
    dispatch("cancelEdit");
  }
</script>

  {#if $cell === "Editing" }
    <div class="editing" on:keydown={keydown} >
      <Multiselect {value} {options}  on:change={ (e) => value = e.detail }/>
      <ActionButton icon = "Checkmark" on:click={ (e) => { e.preventDefault(); e.stopPropagation(); dispatch("submit" ); } }/> 
      <ActionButton icon = "Cancel" on:click={cancelEdit} /> 
    </div>
  {:else}
    <div class="inline-value">
       {#each value as item }
          <div class="item"> {item} </div>
       {/each} 
    </div>
  {/if}

<style>
  .editing {
    display: flex;
    justify-content: center;
  }


  .inline-value { 
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    column-gap: 0.5rem;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }

  .item {
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: darkcyan;
    color: white;
    height: 50%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    max-height: 1.85rem;
  }

  input.inline-edit {
    box-sizing: border-box;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    outline: none;
    background: none;
    color: inherit;
    border: none;
    font: inherit;
    cursor: pointer;
    height: 100%;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>