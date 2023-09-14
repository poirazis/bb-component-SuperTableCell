<script>
  import { createEventDispatcher, onMount } from 'svelte'

  export let value
  export let formattedValue
  export let cellState

  const dispatch = createEventDispatcher()

  let original
  $: inEdit = $cellState == "Editing"

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

<div class="superCell">
  {#if inEdit }
    <input class="inline-edit" type="text" inputmode="numeric" bind:value use:focus on:keydown={keydown} on:blur={submit}/>
  {:else}
    <div class="inline-value"> { formattedValue || value || "" } </div>
  {/if}
</div>

<style>
  .inline-value { 
    flex: auto;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    white-space: nowrap;
  }
  
  input.inline-edit {
    box-sizing: border-box;
    outline: none;
    background: none;
    color: inherit;
    border: none;
    font: inherit;
    cursor: pointer;
    height: 100%;
    width: 100%;
    min-width: none;
    text-align: right;
  }
</style>