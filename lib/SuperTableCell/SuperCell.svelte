<script>
  import { processStringSync }  from "@budibase/string-templates"
  import  fsm  from "svelte-fsm"

  import CellString from "./cells/CellString.svelte";
  import CellLink from "./cells/CellLink.svelte";
  import CellDatetime from "./cells/CellDatetime.svelte";
  import CellBoolean from "./cells/CellBoolean.svelte"
  import CellAttachment from "./cells/CellAttachment.svelte";
  import CellOptions from "./cells/CellOptions.svelte";
  import CellNumber from "./cells/CellNumber.svelte";
  import CellJson from "./cells/CellJSON.svelte";

  export let value 
  export let valueTemplate
  export let fieldSchema
  export let editable
  export let initialState = "View"
  export let lockState
  export let unstyled

  let innerCellState

  export let cellState = fsm( initialState , {
    View: { focus () { return "Focused" }, },
    Hovered: { cancel: () => { return "View" }},
    Focused: { 
      _enter() { if (editable) this.enterEditing.debounce(50) },
      unfocus() { return "View" },
      enterEditing: "Editing",
    },
    Error: { check : "View" },
    Editing: { 
      focus() {},
      unfocus() { if (!innerCellState || $innerCellState == "Closed" ) return "View" },
      lostFocus() { this.unfocus.debounce(10) },
      submit() { if ( value != originalValue ) acceptChange() ; return "View" }, 
      cancel() { value = Array.isArray(originalValue) ? [ ... originalValue ] : originalValue ; return "View" },
    }
  })

  const getCellValue = (value, template) => {
    if (!valueTemplate) {
      return value
    }
    return processStringSync(template, { value })
  }

</script>

{#if fieldSchema.type === "string" || fieldSchema.type === "longform" || fieldSchema.type === "formula"}
  <CellString
    {cellState}
    {fieldSchema}
    {value}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{:else if fieldSchema.type === "array" || fieldSchema.type === "option"  }
  <CellOptions
    bind:editorState={innerCellState}
    {cellState}
    {value}
    {fieldSchema}
    multi={fieldSchema.type == "array"}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{:else if fieldSchema.type === "number" || fieldSchema.type == "bigint" }
  <CellNumber
    bind:editorState={innerCellState}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{:else if fieldSchema.type === "datetime"}
  <CellDatetime
    bind:editorState={innerCellState}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{:else if fieldSchema.type === "link"  }
  <CellLink
    bind:editorState={innerCellState}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{:else if fieldSchema.type === "attachment"  }
  <CellAttachment
    bind:editorState={innerCellState}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{:else if fieldSchema.type === "boolean"  }
  <CellBoolean
    bind:editorState={innerCellState}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{:else if fieldSchema.type === "json"  }
  <CellJson
    bind:editorState={innerCellState}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:blur
  />
{/if}

<style>
  :global(.superCell) {
    flex: auto;
    display: flex;
    align-items: stretch;
    cursor: pointer;
    min-width: 0;
    box-sizing: border-box;
  }

  :global(.superCell.unstyled) {
    flex: auto;
    display: flex;
    align-items: stretch;
    cursor: pointer;
    min-width: 0;
    box-sizing: border-box;
  }
</style>