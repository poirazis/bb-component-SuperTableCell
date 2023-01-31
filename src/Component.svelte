<script>
  import { getContext, createEventDispatcher, onMount } from 'svelte'
    import { bubble } from 'svelte/internal';
  import { fade } from "svelte/transition"

  import CellRelationship from './lib/CellRelationship.svelte';

  const { styleable, builderStore  } = getContext("sdk")
  const component = getContext("component")
  const columnInfo = getContext("SuperTableColumnInfo")
  const tableStore = getContext("tableStore")

  export let value, required = true
  export let valueColor
  export let editable
  export let cellType
  export let viewType

  let autobind = value ?? true 
  $: fontSize = $tableStore?.stylingOptions.rowFontSize ?? 14

  const dispatch = createEventDispatcher()
  let editing = false, original
  let holdColor

  onMount(() => {
    original = value
  })

  function edit() {
    editing = true
    holdColor = $tableStore.stylingOptions.rowFontColor
    $tableStore.stylingOptions.rowFontColor = "var(--spectrum-global-color-gray-100)"
  }

  function submit() {
		if (value != original) {
			dispatch('submit', value)
		}
    editing = false
    $tableStore.stylingOptions.rowFontColor = holdColor
  }

  function keydown(event) {
    if (event.key == 'Escape') {
      event.preventDefault()
      value = original
      editing = false
    }
  }
	
	function focus(element) {
		element.focus()
	}

  $: if (columnInfo && autobind) pickyBinder()

  function pickyBinder()  {
    // Auto bind the value to the Super Column CellValue binding if plaed in one
    if ( $builderStore?.inBuilder ) {
      let binding = "{{ [" + columnInfo?.name + "].[cellValue]" + " }}"
      builderStore?.actions.updateProp("value", binding)
      value = binding
      autobind = false
    }
  }
    // Append Styles Super Table Styling variables
    $: styles = {
    ...$component?.styles,
    normal: {
      ...$component?.styles.normal,
      "--super-table-cell-editable-height": fontSize + 14 + "px"
    },
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div 
  class="super-table-cell" 
  on:click={edit}
  class:in-edit-mode={editing} 
  use:styleable={styles}>
  <!-- Check valid placement  -->
  {#if !columnInfo || !tableStore}
    <p> Super Table Cells need to be placed inside Super Table Column Component </p>
  {:else}
    {#if editable && editing }
      <form on:submit|preventDefault={submit} on:keydown={keydown}>
        <input style:--super-table-row-font-size={fontSize + "px"} class="inlineTextInput" bind:value on:blur={submit} {required} use:focus/>  
      </form>
    {:else if editable}
      {value}
    {:else}
      {#if cellType == "relationship"}
        <CellRelationship {viewType} {value} open={false} />
      {:else}
          {value} 
      {/if}
    {/if}
  {/if}
</div>

<style>

  .super-table-cell {
    display: inline-flex;
    flex-direction: column;
    justify-content: center;
    transition: all 1500ms;
    min-height: var(--super-table-cell-editable-height);

  }
  .super-table-cell.in-edit-mode {

  }

  .inlineTextInput {
    font-size: var(--super-table-row-font-size);
    font-weight: inherit;
    font-style: italic;
    color: inherit;
    border: none;
    background: none;
    outline: none;
    min-width: fit-content !important;
    width: 60px;
    padding: 4px 4px 6px 0px;
    animation: fade-in 330ms ease-in-out;
  }

  .inlineTextInput:focus {
    border: none;
    border-bottom: 1px solid rgb(2, 101, 220);
    color: var(--spectrum-global-color-gray-900);
    outline: none;
  }

  .spectrum-Icon {
    width: 12px;
    height: 12px;
    fill: var(--spectrum-global-color-gray-600);
    display: none;
  }

  @keyframes fade-in {
    0% {
      opacity: 0%;
    }
    100% {
      transform: 100%;
    }
  }
</style>