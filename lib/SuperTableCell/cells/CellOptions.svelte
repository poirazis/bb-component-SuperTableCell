<script>
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { createEventDispatcher ,beforeUpdate } from "svelte"
  import { flip } from 'svelte/animate';

  export let cellState
  export let cellOptions
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

  export let editorState = fsm( "Closed", {
    "*": {
      handleKeyboard( e ) { }
    },
    Open: {  
      toggle() { cellState.closeEditor(); anchor.focus(); return "Closed" },
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
      toggle() { cellState.openEditor(); return "Open" },
      highlightNext () { return "Open" },
    }
  });

  $: value = Array.isArray(value) ? value : value ? [ value ] : []
  $: options = fieldSchema?.constraints?.inclusion || [];
  $: optionColors = fieldSchema?.optionColors || {};
  $: allowNull = !fieldSchema?.constraints?.presence ?? false;
  $: if (allowNull && options.length > 1) options = [ "Clear Selection", ...options ]
  $: inEdit = $cellState == "Editing"
  $: multi = fieldSchema?.type == "array" ?? false
  $: if ( inEdit && anchor && editorState == "Closed" ) anchor?.focus() 

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
      editorState.toggle();
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
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  bind:this={anchor} 
  class="superCell" 
  class:inEdit
  class:unstyled
  class:focused={$cellState == "Focused"}
  style:padding-left={cellOptions?.padding}
  style:padding-right={cellOptions?.padding}
  tabindex="0" 
  on:keydown={handleKeyboard} 
  on:click={ () => { if ( $cellState == "Editing" ) editorState.toggle() } }
  on:blur={() => { cellState.lostFocus() } }
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
      <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-chevron-down"><path d="m6 9 6 6 6-6"/></svg>
    </div>
  {:else if inEdit}
    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-chevron-down"><path d="m6 9 6 6 6-6"/></svg>
  {:else if overflow}
    <div class="overflow" style:background-color={fadeToColor} ></div>
  {/if}
  
  <Popover 
    {anchor} 
    useAnchorWidth={!multi}
    dismissible
    align={"left"} 
    open={ $editorState == "Open" && inEdit }
    on:close={ () => { editorState.toggle() } }
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
                <svg xmlns="http://www.w3.org/2000/svg" 
                  width="16" height="16" viewBox="0 0 24 24" 
                  fill="none" stroke="var(--spectrum-global-color-red-500)" 
                  stroke-width="2" stroke-linecap="round" 
                  stroke-linejoin="round" class="lucide lucide-x">
                    <path d="M18 6 6 18"/><path d="m6 6 12 12"/>
                </svg>
                {option}
              </div>
            </div>
          {:else}
            <div
              class="option"
              class:focused={focusedOptionIdx === idx}
              on:mousedown|preventDefault|stopPropagation={(e) => editorState.toggleOption(idx)}
              on:mouseenter={() => (focusedOptionIdx = idx)}
            >
              <div class="option text">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill={color} stroke={color} stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-square"><rect width="18" height="18" x="3" y="3" rx="2"/></svg>
                {option}
              </div>
              {#if value?.includes(option)}
                <svg xmlns="http://www.w3.org/2000/svg" 
                width="14" height="14" viewBox="0 0 24 24" 
                fill="none" stroke="var(--primaryColor)" 
                stroke-width="2" 
                stroke-linecap="round" 
                stroke-linejoin="round" 
                class="lucide lucide-check">
                  <path d="M20 6 9 17l-5-5"/>
                </svg>
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
    height: 90%;
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