<script>
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte";
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { beforeUpdate } from "svelte"

  export let value;
  export let fieldSchema;
  export let inEdit = false;
  export let multi = false;
  export let useOptionColors = false
  export let defaultOptionColor = "#19647E"
  export let isHovered = false
  export let placeholder = multi ? "Choose options" : "Choose an option";
  export let fadeToColor = "var(--spectrum-global-color-gray-50)"

  export const isOpen = () => { return $editorState == "Open"}
  export const isEmpty = () => { return value.length < 1 }

  let anchor;
  let valueAnchor
  let overflow = false
  let allowNull = fieldSchema.constraints.presence ?? false;
  let focusedOptionIdx = null;

  let editorState = fsm("Closed", {
    Open: { toggle( e ) { if ( e && inEdit ) e.stopPropagation (); return "Closed" } },
    Closed: { toggle ( e ) { if ( e && inEdit ) e.stopPropagation (); return "Open"  } },
  });

  /**
   * Make sure value is always an array
   */
  $: value = multi ? (value ? value : []) : value ? [value] : [];

  $: options = fieldSchema?.constraints?.inclusion || [];
  $: optionColors = fieldSchema?.optionColors || {};
  $: if (!inEdit && $editorState == "Open") editorState.toggle();

  const getOptionColor = (value) => {
    return defaultOptionColor;
  };

  const isOverflown = ( element ) => {
    return element ? element.clientWidth < element.scrollWidth : false
  }

  function toggleOption(option) {
    if (option == "_clearSelection") {
      value = multi ? [] : null;
      editorState.toggle();
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

  beforeUpdate( () => { overflow = isOverflown(valueAnchor); console.log(overflow) } )
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="control" class:inEdit bind:this={anchor} >
  <div class="inline-value" class:inEdit bind:this={valueAnchor} on:click={editorState.toggle} >
    {#if value.length < 1 && inEdit}
      <span class="placeholder">{ placeholder } </span>
    {:else}
      {#each value as val (val)}
        {@const color = optionColors[val] || getOptionColor(val)}
          <div class="item text" style="--color: {color}">
            <span> {val} </span>
          </div>
      {/each}
    {/if}
    {#if overflow }
      <div class="overflow" style:background-color={fadeToColor} />
    {/if}
  </div>

  {#if inEdit}
    <div class="arrow">
      <Icon name="ChevronDown" hoverable on:click={editorState.toggle} />
    </div>
  {/if}

  <Popover on:close={editorState.toggle} {anchor} align={"left"} open={ $editorState == "Open"} >
    <div class="options" on:wheel={(e) => e.stopPropagation()}>
      {#if !allowNull}
        <div
          class="option"
          on:click|stopPropagation={() => toggleOption("_clearSelection")}
        >
          <div class="option text">
            <Icon name="Cancel" size="S" color={"var(--primaryColor)"} />
            None
          </div>
        </div>
      {/if}
      {#each options as option, idx (idx)}
        {@const color = optionColors[option] || getOptionColor(option)}
        <div
          class="option"
          on:click|stopPropagation={() => toggleOption(option)}
          class:focused={focusedOptionIdx === idx}
          on:mouseenter={() => (focusedOptionIdx = idx)}
        >
          <div class="option text">
            <Icon name="LoupeView" size="S" {color} />
            {option}
          </div>
          {#if value?.includes(option)}
            <Icon name="Checkmark" size="S" color="var(--primaryColor)" />
          {/if}
        </div>
      {/each}
    </div>
  </Popover>
</div>

<style>
  .control {
    box-sizing: content-box;
    flex: auto;
    display: flex;
    align-items: stretch;
    cursor: pointer;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    min-width: 0;
  }
  .overflow {
    position: absolute;
    right: 0px;
    top: 20%;
    height: 60%;
    width: calc( 3 * var(--super-table-cell-padding));
    mask-image: linear-gradient(to right, transparent 0%, black 75% );
    mask-mode: alpha;
    z-index: 2;
    transition: all 130ms;
    border-radius: 2px;
  }
  .inline-value {
    position:relative;
    flex-grow: 1;
    flex-shrink: 1;
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 0.5rem;
    overflow: hidden;
  }
  .inline-value.inEdit {
    cursor: pointer;
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
  .multi .text {
    flex: 0 0 auto;
  }
  .arrow {
    display: grid;
    place-items: center;
    z-index: 3;
    background-color: transparent;
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
