<script>
  import { getContext } from "svelte";
  import SuperTableCell from "../lib/SuperTableCell/SuperTableCell.svelte";

  const component = getContext("component");
  const { styleable } = getContext("sdk");
  const columnOptions = getContext("superColumnOptions")

  export let value,
    rowKey,
    required = true;
  export let valueColor;
  export let editable;
  export let cellType;
  export let viewType;

  export let fontColor, fontSize, isBold, isUnderline, isItalic;


  $: cellOptions = {
    color : fontColor
  }
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div use:styleable={$component.styles}>
  {#if !columnOptions}
    <p> 🛑 The Super Table Cell can only be used inside a Super Table Column </p>
  {:else}
    <SuperTableCell 
      {cellOptions}
      {rowKey} 
      {value} 
      fieldSchema={$columnOptions.schema} 
    />
  {/if}
</div>
