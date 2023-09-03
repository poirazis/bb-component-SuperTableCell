<script>
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte";
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { createEventDispatcher, beforeUpdate } from "svelte";

	import { fly } from 'svelte/transition';
  import CellLinkPicker from "./CellLinkPicker.svelte";
  const dispatch = createEventDispatcher();

  export let value = []
  export let inEdit;
  export let fieldSchema;
  export let isHovered = false;

  let anchor;
  let overflow = false
  let regen = false

  let editorState = fsm("Closed", {
    Open: { toggle: "Closed" },
    Closed: { toggle: "Open" },
  });

  const unselectRow = ( val ) => {
    if ( value ) {
      value.splice( value.findIndex ( (e) => e._id === val._id  ), 1 );
      dispatch("change", { value: value} )
    }
  }

  function openPicker(e) {
    e.stopPropagation();
    editorState.toggle();
  }

  beforeUpdate ( () => overflow = anchor ? anchor.clientWidth < anchor.scrollWidth : false )
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div bind:this={anchor} class="control" class:inEdit>
  <div class="inline-value">
    {#if inEdit}
      <Icon name="Add" hoverable size="S" on:click={(e) => openPicker(e)} />
    {/if}

    {#if value}
      {#each value as row }
        <div class="item" >
          {#if inEdit}
            <Icon size="XS" name="Unlink" hoverable color={"var(--primaryColor)"} on:click={ (e) => { e.stopPropagation(); unselectRow(row); } }/>
          {/if}
          {row.primaryDisplay}
        </div>
      {/each}
    {/if}

  </div>

  {#if overflow}
    <div class="overflow" class:wide={isHovered}> 
      {#if isHovered}
        <div class="circle" > {value?.length ?? 0 } </div> 
      {/if}
    </div>
  {/if}
  
  {#if inEdit}
    <Popover {anchor} align = "left" open={$editorState == "Open"} on:close={editorState.toggle}>
      <CellLinkPicker active = { $editorState == "Open" } {value} tableId={fieldSchema.tableId} on:change />
    </Popover>
  {/if}

</div>

<style>
  .control {
    flex: auto;
    display: flex;
    align-items: stretch;
    justify-content: stretch;
    max-height: 2.5rem;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    overflow: hidden;
  }
  .overflow {
    box-sizing: border-box;
    position: absolute;
    right: 1px;
    top: 1px;
    height: calc(100% - 2px);
    display: flex;
    align-items: center;
    justify-content: flex-end;
    width: calc( 4 * var(--super-table-cell-padding));
    color: var(--spectrum-global-color-gray-500);
    background: linear-gradient(to right, transparent 0%, var(--spectrum-global-color-gray-50) 60%);
    transition: all 130ms ease-in-out;
  }
  .wide {
    padding-right: var(--super-table-cell-padding);
    width: calc( 6 * var(--super-table-cell-padding));
    background: linear-gradient(to right, transparent 0%, var(--spectrum-global-color-gray-100) 60%);
  }
  .circle {
    width: 20px;
    height: 20px;
    font-size: 14px;
    font-weight: bold;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    background-color: var(--spectrum-global-color-gray-400);
    color: var(--spectrum-global-color-gray-800);
  }
  .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
  .inline-value {
    flex: 1 1 auto;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    column-gap: 0.5rem;
  }
  .item {
    flex: 0 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: #2d627a;
    color: var(--spectrum-global-color-gray-800);
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    white-space: nowrap;
    gap: 0.5rem;
  }
</style>
