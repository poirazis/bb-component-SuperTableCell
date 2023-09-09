<script>
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte";
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { createEventDispatcher, beforeUpdate } from "svelte";
  import CellLinkPicker from "./CellLinkPicker.svelte";
  const dispatch = createEventDispatcher();

  export let value 
  export let cellState
  export let fieldSchema;
  export let isHovered = false;

  export let editorState= fsm("Closed", {
    Open: { toggle: "Closed" },
    Closed: { toggle: "Open" },
  });

  let anchor;
  let overflow = false
  let regen = false

  $: inEdit = $cellState == "Editing"

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

  const handleKeyboard = ( e ) => {

  }
  beforeUpdate ( () => overflow = anchor ? anchor.clientWidth < anchor.scrollWidth : false )
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  bind:this={anchor} 
  class="superTableCell" 
  class:inEdit
  class:focused={$cellState == "Focused"}
  tabindex="0" 
  on:keydown={handleKeyboard} 
  on:focus={ cellState.focus }
  on:blur={ () => { setTimeout ( cellState.unfocus, 10 ) } }
>
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
      <CellLinkPicker 
        active = { $editorState == "Open" } 
        {value} 
        tableId={fieldSchema.tableId} 
        on:change={(e) => value = e.detail.value }
      />
    </Popover>
  {/if}

</div>

<style>
  .superTableCell {
    flex: auto;
    display: flex;
    align-items: stretch;
    cursor: pointer;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    border: 1px solid transparent;
    min-width: 0;
    box-sizing: border-box;
  }
  .superTableCell.inEdit {
    width: var(--lock-width);
    max-width: var(--lock-width);
    color: var(--spectrum-global-color-gray-900);
    border-color: var(--spectrum-alias-border-color-mouse-focus);
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
  .superTableCell.focused {
    border-color: var(--spectrum-global-color-gray-500);
  }
  .superTableCell:focus {
    outline: none;
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
  .inEdit::before {
    content: "";
    position: absolute;
    top: 1;
    right: 1;
    left: 1;
    bottom: 1;
    filter: brightness(85%);
    background-color: var(--spectrum-global-color-gray-50);
  }
  .inline-value {
    flex: 1 1 auto;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    column-gap: 0.5rem;
    z-index: 1;
  }
  .item {
    flex: 0 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: var(--spectrum-global-color-seafoam-400);
    color: var(--spectrum-global-color-gray-800);
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    white-space: nowrap;
    gap: 0.5rem;
  }
</style>
