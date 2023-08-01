<script>
  import { createEventDispatcher, onMount } from 'svelte'
  import { clickOutsideAction } from "svelte-legos";
  import Switch from "../../../node_modules/@budibase/bbui/src/Form/Core/Switch.svelte"
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte"

  export let value
  export let editable

  const dispatch = createEventDispatcher()
  let editing = false, original

  onMount(() => {
    original = value
  })

  function edit() {
    if (editable) {
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

  function cancelEdit() {
    editing = false;
    dispatch("cancelEdit");
  }
</script>
  {#if !editing}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div   
    class="inline-value"
    on:click={edit} > 
    {#if value}
      <Icon size="XS" name="Checkmark" color={"lime"}/>
    {/if}
  </div>
  {:else}
  <div 
    class="inline-value"
    use:clickOutsideAction
    on:clickoutside={cancelEdit} 
  > 
    <Switch {value} on:change={(e) => value = e.detail}/>
  </div>
  {/if}


<style>
  .inline-value { 
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: var(--super-column-alignment);
    white-space: nowrap;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }
</style>