<script>
  import { createEventDispatcher, onMount } from 'svelte'

  export let value
  export let formattedValue
  export let inEdit

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
    <input class="inline-edit" inputmode="numeric" bind:value use:focus on:keydown={keydown} on:blur={submit}/>
  {:else}
    <div class="inline-value"> { formattedValue || value || "" } </div>
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
    flex: auto;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    white-space: nowrap;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
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
    width: 100%;
    min-width: none;
    text-align: right;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>