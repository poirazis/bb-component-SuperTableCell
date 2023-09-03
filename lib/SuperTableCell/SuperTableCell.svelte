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
  export let isHovered = false;

  export let fontColor, fontSize, isBold, isUnderline, isItalic;
  export let valueColor;

  let originalValue = Array.isArray(value) ? [ ... value ] : value
  let cellAnchor
  let width

  const getCellValue = (value, template) => {
    if (!valueTemplate) {
      return value
    }
    return processStringSync(template, { value })
  }

  const cellState = fsm( "View", {
    View: { 
      focus: () => { return editable ? "Editing" : "Focused" } },
    Hovered: { cancel: () => { return "View" }},
    Focused: { cancel() { return "View" }},
    Error: { check : "View" },
    Editing: { 
      _enter() { width = cellAnchor ? cellAnchor.clientWidth - 2 : "auto" },
      submit() { if ( value != originalValue ) acceptChange() ; return "View" }, 
      cancel() { value = Array.isArray(originalValue) ? [ ... originalValue ] : originalValue ; return "View" }}
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
    originalValue = Array.isArray(value) ? [ ... value ] : value
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
    if ( $cellState == "Editing")
      if ( submitOn != "onBlur" ) 
        cellState.cancel();
      else 
        cellState.submit();
    else 
      cellState.cancel()
  }

  function handleFocus () {
    if ( $cellState != "View" ) 
      cellState.cancel()
    else
      cellState.focus()
  }

  function handleChange ( e ) {
    value = e.detail.value
    console.log("Change Received", value )
  }

</script>

<!-- Will Reder Different CellTypes depending on the column type or manual override -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  class="superTableCell"
  class:focused={ $cellState == "Focused" }
  class:inEdit={ $cellState == "Editing" }
  tabindex="0"
  bind:this={cellAnchor}
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
      {width}
      {value}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "formula"}
    <CellString
      inEdit = { false }
      {value}
      {width}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "number"}
    <CellNumber
      inEdit = { $cellState == "Editing"}
      {value}
      {width}
      formattedValue = { getCellValue(value, valueTemplate) }
      on:change={handleChange}
    />
  {:else if fieldSchema.type === "bigint"}
    <CellNumber
      inEdit = { $cellState == "Editing"}
      {value}
      {width}
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
      inEdit = { $cellState == "Editing" }
      {value}
      {fieldSchema}
      {isHovered}
      on:change={handleChange}
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
    flex: 1 1 auto;
    display: flex;
    justify-content: stretch;
    align-items: stretch;
    background: transparent;
    color: var(--super-column-color);
    border-width: 1px;
    border-style: solid;
    border-color: transparent;
    overflow: hidden;
  }
  .inEdit {
    color: var(--spectrum-global-color-gray-900);
    border-color: var(--spectrum-alias-border-color-mouse-focus);
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
  .focused {
    border-color: var(--spectrum-global-color-gray-500);
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }

</style>