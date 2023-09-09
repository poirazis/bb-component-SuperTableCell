<script>
  import { getContext } from "svelte";
  import { processStringSync }  from "@budibase/string-templates"
  import { clickOutsideAction } from "svelte-legos";
  import fsm from "svelte-fsm";

  import CellString from "./cells/CellString.svelte";
  import CellLink from "./cells/CellLink.svelte";
  import CellDatetime from "./cells/CellDatetime.svelte";
  import CellBoolean from "./cells/CellBoolean.svelte"
  import CellAttachment from "./cells/CellAttachment.svelte";
  import CellOptions from "./cells/CellOptions.svelte";
  import CellNumber from "./cells/CellNumber.svelte";

  const tableDataChangesStore = getContext("tableDataChangesStore");

  export let value
  export let rowKey
  export let editable
  export let fieldSchema
  export let valueTemplate
  export let submitOn = "onEnter"
  export let isHovered = false;

  let originalValue = Array.isArray(value) ? [ ... value ] : value
  let width
  let editorState

  const getCellValue = (value, template) => {
    if (!valueTemplate) {
      return value
    }
    return processStringSync(template, { value })
  }

  const cellState = fsm( "View", {
    "*" : { 
      focus () { return "Focused" },
      unfocus () { return "View" },
      handleKeyboard ( e ) {
        if ( e.key == "Escape" )
          this.unfocus()
      }
    },
    View: { },
    Hovered: { cancel: () => { return "View" }},
    Focused: { 
      _enter() { if (editable) this.enterEditing.debounce(50) },
      enterEditing: "Editing"
    },
    Error: { check : "View" },
    Editing: { 
      focus() { },
      unfocus() { 
        if ( editorState && $editorState == "Closed" ) 
          return "View" 
        else if ( editorState == undefined)
          return "View"
      },
      openEditor() { if ( editorState ) editorState.open() },
      closeEditor() { if ( editorState ) editorState.close() },
      submit() { if ( value != originalValue ) acceptChange() ; return "View" }, 
      cancel() { value = Array.isArray(originalValue) ? [ ... originalValue ] : originalValue ; return "View" },
    }
  })

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


  function handleChange ( e ) {
    value = e.detail.value
    console.log("Change Received", value )
  }

  $: console.log("Cell ", $cellState)
</script>

<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  class="superTableCellWrapper" 
  on:keydown={cellState.handleKeyboard}
  on:click={cellState.focus}
  use:clickOutsideAction
  on:clickoutside={ () => setTimeout( cellState.unfocus, 10 ) }
>
  {#if fieldSchema.type === "string" || fieldSchema.type === "longform" || fieldSchema.type === "formula"}
    <CellString
      bind:value
      {cellState}
      formattedValue = { getCellValue(value, valueTemplate) }
      {width}
    />
  {:else if fieldSchema.type === "number" || fieldSchema.type === "bigint"}
    <CellNumber
      inEdit = { $cellState == "Editing"}
      {width}
      formattedValue = { getCellValue(value, valueTemplate) }
      bind:value
    />
  {:else if fieldSchema.type === "array" || fieldSchema.type === "options" }
    <CellOptions
      bind:value
      bind:editorState
      {cellState}
      {fieldSchema}
      fadeToColor={ isHovered ? "var(--spectrum-global-color-gray-100)" : "var(--spectrum-global-color-gray-50)" }
      {isHovered} 
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
      bind:value
      bind:editorState
      {cellState}
      {fieldSchema}
      isHovered={false}
    />
  {:else if fieldSchema.type === "boolean"}
    <CellBoolean
      inEdit = { $cellState == "Editing"}
      on:change={handleChange}
      {value}
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
  .superTableCellWrapper {
    flex: auto;
    display: flex;
    align-items: stretch;
    overflow: hidden;
  }
  
</style>