<script>
  import { getContext } from "svelte";
  import { SuperTableCell } from "../bb_super_components_shared/src/lib"

  const component = getContext("component");
  const { styleable } = getContext("sdk");

  const columnOptions = getContext("superColumnOptions")
  const tableHoverStore = getContext("tableHoverStore");

  export let value = []
  export let rowKey
  export let required = true;
  export let color;
  export let editable;
  export let cellType;
  export let viewType;

  export let fontColor, fontSize, isBold, isUnderline, isItalic;

  let parsedValue = []

  $: isHovered = false
  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "flex": columnOptions ? "1 1 auto" : null,
      "display": columnOptions ? "flex" : null,
      "align-items": columnOptions ? "stretch" : null
    }
  }

  $: if ( $columnOptions?.schema?.relationshipType && value ) {
    parsedValue = JSON.parse(value) ?? []
  } else {
    parsedValue = value
  }
</script>

<div use:styleable={$component.styles}>
  {#if columnOptions}
    <SuperTableCell 
      {rowKey} 
      value={parsedValue}
      {editable} 
      fieldSchema={$columnOptions.schema}
      {isHovered} 
      columnOptions={{
        ...$columnOptions,
        "color" : color ? color : $columnOptions.color,
        "canEdit" : true
        }}
    /> 
  {:else}
    <p> Super Table Cell can only be used inside a Super Table Column </p>
  {/if}
</div>