<script>
  import { getContext } from "svelte";
  import fsm from "svelte-fsm";
  import clickOutside from "../../../node_modules/@budibase/bbui/src/Actions/click_outside"
  
  import { processStringSync } from "@budibase/string-templates"

  import CellString from "./cells/CellString.svelte";
  import CellLink from "./cells/CellLink.svelte";
  import CellDatetime from "./cells/CellDatetime.svelte";
  import CellBoolean from "./cells/CellBoolean.svelte"
  import CellAttachment from "./cells/CellAttachment.svelte";
  import CellOptions from "./cells/CellOptions.svelte";
  import CellNumber from "./cells/CellNumber.svelte";

  const columnContext = getContext("columnContext");
  const tableDataChangesStore = getContext("tableDataChangesStore");

  export let value
  export let rowKey
  export let editable
  export let fieldSchema
  export let valueTemplate
  export let submitOn = "onEnter"

  export let fontColor, fontSize, isBold, isUnderline, isItalic;
  export let valueColor;

  let originalValue = value
  let tableCell
  let width

  const getCellValue = (value, template) => {
    if (!valueTemplate) {
      return value
    }
    return processStringSync(template, { value })
  }

  /**
   * Set up Cell state machine.
   */
  const cellState = fsm( "View", {
    View: { focus() { if (editable) {return "Editing"} else {return "Focused"} } },
    Focused: { unfocus: "View"},
    Error: { check : "View" },
    Editing: { 
      _enter() { width = tableCell.clientWidth - 2 },
      submit() { if ( value != originalValue ) acceptChange() ; return "View" }, 
      cancel() { value = originalValue; return "View" }}
  } )

  function acceptChange ( ) { 
    let newDataChange = {
      rowID: rowKey,
      field: fieldSchema.name,
      originalValue: originalValue,
      newValue: value
    };

    // Get all other dataChanges, removing any previous changes of ours
    let dataChanges = $tableDataChangesStore.filter(
      (v) => v.rowKey != rowKey || v.field != fieldSchema.name
    );

    $tableDataChangesStore = [...dataChanges, newDataChange];
  }

  function handleKeyboard ( e ) {
    if ( e.key == "Escape" ) {
      cellState.cancel();
    } else if (e.key == "Enter" ) {
      cellState.submit();
    }
  }

  function handleUnfocus () {
    if ( submitOn != "onBlur" ) 
      cellState.cancel();
    else 
      cellState.submit();
  }

  function handleFocus () {
    if ( $cellState != "View") 
      cellState.cancel()
    else
      cellState.focus()
  }

  function handleChange ( e ) {
    value = e.detail.value
  }
</script>

<!-- Will Reder Different CellTypes depending on the column type or manual override -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  class="superTableCell"
  class:inEdit={$cellState === "Editing"}
  tabindex="0"
  bind:this={tableCell}
  on:click={ handleFocus } 
  on:keydown={ handleKeyboard }
  use:clickOutside={ handleUnfocus }
>
  {#if fieldSchema.type === "string"}
    <CellString
      inEdit = { $cellState == "Editing"}
      {width}
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={ handleChange }
    />
  {:else if fieldSchema.type === "longform"}
    <CellString
      inEdit = { $cellState == "Editing"}
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "formula"}
    <CellString
      inEdit = { false }
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "number"}
    <CellNumber
      inEdit = { $cellState == "Editing"}
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "bigint"}
    <CellNumber
      inEdit = { $cellState == "Editing"}
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "datetime"}
    <CellDatetime
      inEdit = { $cellState == "Editing"}
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "link"}
    <CellLink
      inEdit = { $cellState == "Editing"}
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
      {fieldSchema}
    />
  {:else if fieldSchema.type === "boolean"}
    <CellBoolean
      inEdit = { $cellState == "Editing"}
      on:change={handleChange}
      {value}
    />
  {:else if fieldSchema.type === "array"}
    <CellOptions
      on:change={handleChange}
      inEdit = { $cellState == "Editing" }
      {value}
      multi
      {fieldSchema}
    />
  {:else if fieldSchema.type === "options"}
    <CellOptions
      on:change={handleChange}
      inEdit = { $cellState == "Editing" }
      {value}
      {fieldSchema}
    />
  {:else if fieldSchema.type === "attachment"}
    <CellAttachment
      {cellState}
      {editable}
      {value}
    />
  {/if}
</div>

<style>
  .superTableCell {
    display: flex;
    justify-content: stretch;
    align-items: stretch;
    background: transparent;
    color: var(--super-column-color);
    width: 100%;
    height: 100%;
  }
  .inEdit {
    border-width: 1px;
    border-style: solid;
    border-color: var(--spectrum-alias-border-color-mouse-focus);
  }
</style>