<script>
  import { getContext, createEventDispatcher, onMount } from 'svelte'
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
  export let fontSize

  $: _fontSize = fontSize === "inherit" ? $tableStore.fontSize : fontSize

  const dispatch = createEventDispatcher()
  let editing = false, original

  onMount(() => {
    original = value
  })

  function edit() {
    editing = true
  }

  function submit() {
		if (value != original) {
			dispatch('submit', value)
		}
    editing = false
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

  function initialize ( ) {
    // Auto bind the value to the Super Column CellValue binding
    if ( $builderStore?.inBuilder && !value) {
      let binding = "{{ [" + columnInfo.name + "].[cellValue]" + " }}"
      value = binding
      builderStore.actions.updateProp("value", binding)
    }
  }

  onMount (() => initialize ())
</script>

<div use:styleable={$component.styles}>

    {#if !columnInfo}
      <p> Super Table Cells need to be placed inside Super Table Column Component </p>
    {:else}


  {#if editable && editing }
      <form on:submit|preventDefault={submit} on:keydown={keydown}>
        <input class="inlineTextInput" bind:value on:blur={submit} {required} use:focus/>
        <div class="spectrum-Textfield spectrum-Textfield--quiet">
          <input type="text" placeholder="Enter your name" name="field" bind:value class="spectrum-Textfield-input">
        </div>    
      </form>
  {:else if editable}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div
      style:--valueColor={valueColor || "--primaryColor"} 
      class="value" on:click={edit}>
      {value}			
			<svg class="spectrum-Icon" focusable="false" aria-hidden="true">
				<use xlink:href="#spectrum-icon-18-Edit" />
			</svg>
    </div>
  {:else}
    {#if cellType == "relationship"}
      <CellRelationship {viewType} {value} open={false} />
    {:else}
      <div class="spectrum-Body" 
        style:white-space= { "nowrap"}
        style:font-size={ "var(--st-row-font-size)" }
        style:color={ "var(--spectrum-table-cell-text-color, var(--spectrum-alias-text-color))" }>
        {value}			
      </div> 
    {/if}

  {/if}

  {/if}
</div>

<style>

  .inlineTextInput {
    border: none;
    background: none;
    outline: none;
    width: 100%;
    padding: 4px 12px;
  }

  .inlineTextInput:focus {
    border: none;
    border-bottom: 1px solid var(--spectrum-global-color-gray-500);
    border-radius: 2px;
    background-color: var(--spectrum-global-color-gray-200);;
    color: var(--spectrum-global-color-gray-900);
    outline: none;

  }

  .spectrum-Icon {
    width: 12px;
    height: 12px;
    fill: var(--spectrum-global-color-gray-600);
    display: none;
  }
</style>