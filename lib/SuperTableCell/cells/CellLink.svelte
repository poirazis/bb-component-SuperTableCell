<script>
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte";
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { createEventDispatcher, beforeUpdate } from "svelte";
  import { flip } from "svelte/animate"
  import CellLinkPicker from "./CellLinkPicker.svelte";
  const dispatch = createEventDispatcher();

  export let value 
  export let cellState
  export let fieldSchema;
  export let isHovered;
  export let placeholder
  export let fadeToColor = "var(--spectrum-global-color-gray-50)"
  export let unstyled

  export let editorState=fsm("Closed", {
    Open: { toggle: "Closed" },
    Closed: { toggle: "Open" },
  });

  let anchor;
  let valueAnchor
  let overflow

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
    if ( e.keyCode == 32 ) {
      e.preventDefault();
      e.stopPropagation();
      editorState.toggle();
    }
  }
  beforeUpdate ( () => { 
    overflow = valueAnchor ? valueAnchor.clientWidth != valueAnchor.scrollWidth : undefined
  } )
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  bind:this={anchor} 
  class="superCell"
  class:inEdit
  class:unstyled
  class:focused={$cellState == "Focused"}
  tabindex="0" 
  on:keydown={handleKeyboard} 
>
  <div bind:this={valueAnchor} class="inline-value">
    {#if value?.length < 1 && inEdit}
      <span class="placeholder">{ placeholder } </span>
    {:else if value?.length > 0}
      {#each value as val (val)}
        <div animate:flip={{ duration: 130 }} class="item text">
          <span> {val.primaryDisplay} </span>
        </div>
      {/each}
    {/if}
  </div>

  {#if overflow}
    <div class="overflow" class:inEdit={inEdit || isHovered} style:background-color={ fadeToColor } >
      {#if inEdit}
        <Icon name="Add" hoverable size="S" on:click={(e) => openPicker(e)} />
      {:else if isHovered}
        <div class="circle"> { value?.length } </div>
      {/if}
    </div>
  {:else}
    {#if inEdit}
      <Icon name="Add" hoverable size="S" on:click={(e) => openPicker(e)} />
    {/if}
  {/if}
</div>

<Popover 
  {anchor} 
  dismissible
  align={"left"} 
  open={ $editorState == "Open" } 
  on:close={ editorState.toggle }
  >
  <CellLinkPicker {value} tableId={fieldSchema.tableId} />
</Popover>

<style>
  .circle {
    width: 22px;
    height: 22px;
    font-size: 15px;
    font-weight: bold;
    display: flex;
    align-items: center;
    align-self: center;
    justify-content: center;
    border-radius: 50%;
    color: var(--spectrum-global-color-gray-600);
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
    background-color: #2479BB;
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    white-space: nowrap;
    gap: 0.5rem;
  }
</style>
