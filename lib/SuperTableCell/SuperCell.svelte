<script>
    /**
   * The complete set of options that can be passed to a Super Cell.
   * @typedef {Object} cellOptions
   * @property {Object} schema - The schema of the Cell ( as per Budibase Field Schema ). if Not set, the Cell will render as String
   * @property {string} mode - Can be Field / TableCell / or Unstyled
   * @property {string} state - The State of the Cell. Can be View / Edit / Disabled 
   * @property {string} placeholder - The Cell Placeholder Text
   * @property {string} align - Horizontal Alignment
   * @property {string} color - Use Font Color
   * @property {string} weight - Use Font Weight
   * @property {string} bgColor - The Background Color
   * @property {string} padding - The padding to be applied to the Cell
   * @property {boolean} hovered - To enter hovered state
   */

  import { getContext , createEventDispatcher } from "svelte";
  import fsm from "svelte-fsm"

  import CellString from "./cells/CellString.svelte";
  import CellLink from "./cells/CellLink.svelte";
  import CellDatetime from "./cells/CellDatetime.svelte";
  import CellBoolean from "./cells/CellBoolean.svelte"
  import CellAttachment from "./cells/CellAttachment.svelte";
  import CellOptions from "./cells/CellOptions.svelte";
  import CellNumber from "./cells/CellNumber.svelte";
  import CellJson from "./cells/CellJSON.svelte";

  const { processStringSync } = getContext("sdk");
  const dispatch = createEventDispatcher();

  export let value 
  export let valueTemplate
  export let fieldSchema
  export let editable
  export let initialState = "View"
  export let lockState = false
  export let unstyled
  export let isHovered
  export let placeholder
  export let multi = true

  /** @type {cellOptions} */
  export let cellOptions

  export let cellState = fsm( initialState , {
    "*": {
      goTo( state ) { return state }
    },
    View: { 
      focus () { 
        return editable ? "Editing" : "Focused" 
      }
    },
    Hovered: { cancel: () => { return "View" }},
    Focused: { 
      unfocus() { return lockState ? initialState : "View" },
    },
    Error: { check : "View" },
    Editing: { 
      unfocus() { return lockState ? initialState : "View" },
      lostFocus() { 
        dispatch("blur", {} );
        return lockState ? initialState : "View" },
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
    {cellOptions}
    {cellState}
    {fieldSchema}
    {placeholder}
    {value}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
    on:focus
    on:blur
  />
{:else if fieldSchema.type === "array" || fieldSchema.type === "options"  }
  <CellOptions
    {cellState}
    {cellOptions}
    {value}
    {fieldSchema}
    multi={fieldSchema.type == "array" && multi}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
  />
{:else if fieldSchema.type === "number" || fieldSchema.type == "bigint" }
  <CellNumber
    {cellState}
    {cellOptions}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
  />
{:else if fieldSchema.type === "datetime"}
  <CellDatetime
    {cellState}
    {cellOptions}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
  />
{:else if fieldSchema.type === "link"  }
  <CellLink
    {cellState}
    {cellOptions}
    {value}
    {fieldSchema}
    {unstyled}
    {isHovered}
    on:change
  />
{:else if fieldSchema.type === "attachment"  }
  <CellAttachment
    {cellOptions}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
  />
{:else if fieldSchema.type === "boolean"  }
  <CellBoolean
    {cellOptions}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
  />
{:else if fieldSchema.type === "json"  }
  <CellJson
    {cellOptions}
    {cellState}
    {value}
    {fieldSchema}
    formattedValue = { getCellValue(value, valueTemplate) }
    {unstyled}
    on:change
  />
{/if}

<style>
  :global(.superCell) {
    flex: auto;
    min-width: 80px;
    display: flex;
    cursor: pointer;
    position: relative;
    z-index: 1;
    border: 1px solid var(--spectrum-global-color-gray-300);
    background-color: var(--spectrum-global-color-blue-100);
    padding: 0.3rem 0.85rem;
  }
  :global(.superCell.inEdit) {
    border: 1px solid var(--spectrum-global-color-gray-300);
    background-color: var(--spectrum-global-color-gray-50);
  }
  :global(.superCell.unstyled) {
    border: unset;
    background-color: unset;
    padding: unset;
  }
  :global(.superCell.unstyled.inEdit ) {
    border: unset;
    background-color: unset;
    padding: unset;
  }
  :global(.superCell:focus) {
    outline: none;
  }
  :global(.overflow) {
    position: absolute;
    right: var(--super-table-cell-padding);
    top: 10%;
    height: 90%;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    width: calc( 2 * var(--super-table-cell-padding));
    mask-image: linear-gradient(to right, transparent 0%, black 65% );
    mask-mode: alpha;
    z-index: 1;
    transition: all 130ms;
  }
  :global(.overflow.inEdit) {
    width: calc( 4 * var(--super-table-cell-padding));
    mask-image: linear-gradient(to right, transparent 0%, black 50% );
    border: 1px solid var(--spectrum-global-color-gray-300);
    background-color: var(--spectrum-global-color-gray-50);
  }

  :global(.superCell .lucide) {
    height: 100%;
    display: flex;
    align-items: center;
  }
</style>