<script>
  import { getContext , onMount } from 'svelte'
  import CellString from './lib/CellString.svelte';

  const component = getContext("component")
  const { styleable, builderStore  } = getContext("sdk")
  const columnContext = getContext("columnContext")
  const tableDataChangesStore = getContext("tableDataChangesStore")
  
  export let value, rowKey, required = true
  export let valueColor
  export let editable
  export let cellType
  export let viewType

  export let fontColor, fontSize, isBold, isUnderline, isItalic 

  let autobind = true
  let originalValue

  // Append Styles Super Table Styling variables
  $: styles = {
    ...$component?.styles,
    normal: {
      ...$component?.styles.normal,
      ...generateStylingVariables()
    },
  };

  // Component Function Definitions 
  function pickyBinder()  {
    // Auto bind the value to the Super Column CellValue binding if plaed in one
    if ( $builderStore?.inBuilder ) {
      let valueBinding = "{{ [" + columnContext?.columnID + "].[cellValue]" + " }}"
      builderStore?.actions.updateProp("value", valueBinding)
      let keyBinding = "{{ [" + columnContext?.columnID + "].[rowKey]" + " }}"
      builderStore?.actions.updateProp("rowKey", keyBinding)
      autobind = false
    }
  }

  function generateStylingVariables () {
    let styles = {}
    if ( fontColor ) styles["--spectrum-table-m-regular-cell-text-color"] = fontColor
    if ( isBold ) styles["--spectrum-table-m-regular-cell-text-weight"] = 700
    if ( isItalic ) styles["--spectrum-table-m-regular-cell-text-style"] = "italic"
    if ( isUnderline ) styles["--spectrum-table-m-regular-cell-text-decoration"] = "underline"
    return styles
  }

  // Event Handlers 
  function handleEnterEdit () {
    console.log("Editing")
  }

  function handleSubmit ( event ) {
    let newDataChange = {
				rowKey: rowKey,
				field: columnContext?.columnField,
				originalValue: originalValue,
				newValue: event.detail.newValue
			}

    // Get all other dataChanges, removing any previous changes of ours
    let dataChanges = $tableDataChangesStore.filter( v => v.rowKey != rowKey || v.field != columnContext?.columnField)

    $tableDataChangesStore = [...dataChanges, newDataChange]
  }

  function handleCancelEdit () {
    console.log("Editing Cancelled")
  }

  onMount( () => { 
      if (columnContext && !value && autobind) pickyBinder() 
      originalValue = value
    } )
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="super-table-cell" use:styleable={styles}>
  <!-- Check valid placement  -->
  {#if !columnContext}
    <p> Super Table Cells need to be placed inside Super Table Column Components </p>
  {:else} 
    <CellString on:enterEdit={handleEnterEdit} on:submit={handleSubmit} on:cancelEdit={handleCancelEdit} {value} {editable} />
  {/if}
</div>

<style>
  .super-table-cell {
    display: flex;
    flex-direction: row;
    justify-content: stretch;
    max-width: 350px !important;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: var(--spectrum-table-m-regular-cell-text-color);
    font-weight: var(--spectrum-table-m-regular-cell-text-weight);
    font-style: var(--spectrum-table-m-regular-cell-text-style);
    text-decoration: var(--spectrum-table-m-regular-cell-text-decoration);
    width: 100%;
  }

</style>