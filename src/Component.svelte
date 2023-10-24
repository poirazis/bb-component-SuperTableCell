<script>
  import { getContext } from "svelte";
  import SuperTableCell from "../lib/SuperTableCell/SuperTableCell.svelte";

  const component = getContext("component");
  const { styleable } = getContext("sdk");

  const columnOptions = getContext("superColumnOptions")
  const tableHoverStore = getContext("tableHoverStore");

  export let value,
    rowKey,
    required = true;
  export let color;
  export let editable;
  export let cellType;
  export let viewType;

  export let fontColor, fontSize, isBold, isUnderline, isItalic;

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
</script>

<div use:styleable={$component.styles}>
  {#if columnOptions}
    <SuperTableCell 
      {rowKey} 
      {value} 
      {editable} 
      fieldSchema={$columnOptions.schema}
      {isHovered} 
      columnOptions={{
        ...$columnOptions,
        "color" : color ? color : $columnOptions.color
        }}
    /> 
  {:else}
    <p> Super Table Cell can only be used inside a Super Table Column </p>
  {/if}
</div>