<script>
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte";
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { onMount, createEventDispatcher ,beforeUpdate } from "svelte"
  import { flip } from 'svelte/animate';

  export let cellState
  export let value;
  export let fieldSchema;

  export let useOptionColors = false
  export let defaultOptionColor = "#19647E"
  export let isHovered = false
  export let placeholder = multi ? "Choose options" : "Choose an option";
  export let fadeToColor = "var(--spectrum-global-color-gray-50)"
  export let optionIcon = "LoupeView"
  export let unstyled

  const dispatch = createEventDispatcher()

  let anchor;
  let valueAnchor
  let overflow = true
  let focusedOptionIdx = undefined;
  let lockWidth

  export let editorState = fsm( "Closed", {
    "*": {
      handleKeyboard( e ) { }
    },
    Open: {  
      _enter() { cellState.openEditor() },
      _exit() { cellState.closeEditor() },
      close() { return "Closed" },
      toggle() { return "Closed" },
      toggleOption ( idx ) { 
        toggleOption(options[idx]); 
        dispatch("change", value )
        if (!multi) this.close.debounce(100)
      },
      highlightNext () { 
        if ( focusedOptionIdx == options.length - 1 )
          focusedOptionIdx = 0
        else if ( focusedOptionIdx != undefined )
          focusedOptionIdx = focusedOptionIdx + 1; 
        else 
          focusedOptionIdx = 0;
      },
      highlightPrevious () { 
        if ( focusedOptionIdx == 0 ) 
          focusedOptionIdx = options.length - 1  
        else if ( focusedOptionIdx == undefined )
          focusedOptionIdx = options.length - 1
        else
          focusedOptionIdx = focusedOptionIdx -1;
      },
    },
    Closed: {
      toggle() { return "Open" },
      open() { return "Open" },
      highlightNext () { return "Open" },
    }
  });

  $: value = Array.isArray(value) ? value : value ? [ value ] : []
  $: options = fieldSchema?.constraints?.inclusion || [];
  $: optionColors = fieldSchema?.optionColors || {};
  $: allowNull = !fieldSchema?.constraints?.presence ?? false;
  $: if (allowNull && options.length > 1) options = [ "Clear Selection", ...options ]
  $: inEdit = $cellState == "Editing" || $cellState == "EditingWithEditor"
  $: multi = fieldSchema?.type == "array" ?? false

  const getOptionColor = (value) => {
    return defaultOptionColor;
  };

  const handleKeyboard = ( e ) => {
    if ( $editorState == "Open" ) {  
      if ( e.key == "ArrowDown" ) editorState.highlightNext( e.preventDefault() );
      if ( e.key == "ArrowUp" ) editorState.highlightPrevious();
      if ( e.key == "Escape" ) editorState.close( e.stopPropagation() ) ;
      if ( e.key == "Enter" ) editorState.close( e.stopPropagation() ) ;
      if ( e.key == "Tab" ) editorState.close();
      if ( e.keyCode == 32 ) {
        e.preventDefault();
        e.stopPropagation();
        focusedOptionIdx != undefined ? editorState.toggleOption(focusedOptionIdx) : editorState.toggle();
      }  
    } else if ( $cellState == "Editing") {
      if ( e.keyCode == 32 ) editorState.toggle( e.preventDefault() );
    }
  }

  const toggleOption = ( option ) => {
    if (option == "Clear Selection") {
      value = multi ? [] : null;
      return;
    }

    if (!multi) {
      value = [option];
    } else {
      if (value.includes(option)) {
        value.splice(value.indexOf(option), 1);
        value = value;
      } else {
        value = [...value, option];
      }
    }
  }

  beforeUpdate ( () => { 
    overflow = valueAnchor ? valueAnchor.clientWidth != valueAnchor.scrollWidth : undefined
  } )
  onMount( () => { 
    lockWidth = anchor.clientWidth 
    if ( $cellState != "View" )
      anchor.focus();
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
  tabindex="-1" 
  on:keydown={handleKeyboard} 
>
  <div bind:this={valueAnchor} class="inline-value" >
    {#if value.length < 1 && inEdit}
      <span class="placeholder">{ placeholder } </span>
    {:else if value.length > 0}
      {#each value as val (val)}
        {@const color = optionColors[val] || getOptionColor(val)}
          <div animate:flip={{ duration: 130 }} class="item text" style="--color: {color}">
            <span> {val} </span>
          </div>
      {/each}
    {/if}
  </div>

  {#if overflow && inEdit}
    <div class="overflow" class:inEdit style:background-color={ fadeToColor } >
      <Icon name="ChevronDown" hoverable on:click={editorState.open} />
    </div>
  {:else if inEdit}
    <Icon name="ChevronDown" hoverable on:click={editorState.open} />
  {:else if overflow}
    <div class="overflow" style:background-color={fadeToColor} ></div>
  {/if}
  
  <Popover 
    {anchor} 
    useAnchorWidth={!multi}
    dismissible
    align={"left"} 
    open={ $editorState == "Open" } 
    on:close={ editorState.close }
  >
    <div class="options" on:wheel={(e) => e.stopPropagation()}>
      {#if options.length < 1}
        No Available Options
      {:else}
        {#each options as option,idx (idx)}
        {@const color = optionColors[option] || getOptionColor(option)}
          {#if (option == "Clear Selection") }
            <div
              class="option"
              style:color={"var(--primaryColor)"}
              class:focused={focusedOptionIdx === idx}
              on:mouseenter={() => (focusedOptionIdx = idx)}
              on:click|preventDefault={(e) => editorState.toggleOption(idx)}
            >
              <div class="option text">
                <Icon name={"Cancel"} size="S" color="var(--spectrum-global-color-red-500)" />
                {option}
              </div>
            </div>
          {:else}
            <div
              class="option"
              class:focused={focusedOptionIdx === idx}
              on:click|preventDefault|stopPropagation={(e) => editorState.toggleOption(idx)}
              on:mouseenter={() => (focusedOptionIdx = idx)}
            >
              <div class="option text">
                <Icon name={optionIcon} size="S" {color} />
                {option}
              </div>
              {#if value?.includes(option)}
                <Icon name="Checkmark" size="S" color="var(--primaryColor)" />
              {/if}
            </div>
          {/if}
        {/each}
      {/if}
    </div>
  </Popover>
</div>

<style>
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
  .inline-value .placeholder {
    font-style: italic;
    white-space: nowrap;
    text-overflow: ellipsis;
    color: var(--spectrum-global-color-gray-600);
  }
  .item {
    flex: none;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: var(--color);
    color: var(--spectrum-global-color-gray-800);
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    max-height: 1.85rem;
  }
  .text {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .options {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    overflow-y: auto;
    padding: 0.3rem;
    gap: 0rem;
  }
  .option {
    padding: 0.15rem;
    display: flex;
    gap: 0.3rem;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
  }
  .option:hover,
  .option.focused {
    background-color: var(--spectrum-global-color-gray-200);
    border-radius: 4px;
  }
</style>