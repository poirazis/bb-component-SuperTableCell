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
  export let isHovered;
  export let fadeToColor

  export let editorState= fsm("Closed", {
    Open: { toggle: "Closed" },
    Closed: { toggle: "Open" },
  });

  let anchor;
  let valueAnchor
  let overflow = false

  $: inEdit = $cellState == "Editing"

  const unselectRow = ( val ) => {
    if ( value ) {
      value.splice( value.findIndex ( (e) => e._id === val._id  ), 1 );
      value = value
    }
  }

  function openPicker(e) {
    e.stopPropagation();
    editorState.toggle();
  }

  const handleKeyboard = ( e ) => {

  }
  beforeUpdate ( () => overflow = valueAnchor ? valueAnchor.clientWidth < valueAnchor.scrollWidth : false )
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
  <div bind:this={valueAnchor} class="inline-value">
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

    {#if overflow }
      <div class="overflow" style:background-color={fadeToColor} >
        {#if isHovered}
          <div class="circle">{value.length}</div>
        {/if}
      </div>
    {/if}

  </div>
  
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
    z-index: 3;
  }
  .inline-value {
    position: relative;
    flex: 1 1 auto;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    column-gap: 0.5rem;
    z-index: 1;
    overflow: hidden;
  }
  .item {
    display: flex;
    align-items: center;
    border-radius: 4px;
    background-color: var(--super-table-relItem-bg-color);
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    white-space: nowrap;
    gap: 0.5rem;
  }
</style>
