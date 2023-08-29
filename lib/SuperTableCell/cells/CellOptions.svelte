<script>
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte";
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let value;
  export let fieldSchema;
  export let inEdit = false;
  export let multi = false;
  export let placeholder = multi ? "Choose some options" : "Choose an option";

  let anchor;
  let allowNull = fieldSchema.constraints.presence ?? false;

  let editorState = fsm("Closed", {
    Open: { toggle: "Closed" },
    Closed: { toggle: "Open" },
  });

  let focusedOptionIdx = null;

  /**
   * Make sure value is always an array
   */
  $: value = multi ? (value ? value : []) : value ? [value] : [];

  $: options = fieldSchema?.constraints?.inclusion || [];
  $: optionColors = fieldSchema?.optionColors || {};
  $: if (!inEdit && $editorState == "Open") editorState.toggle();

  const getOptionColor = (value) => {
    return "darkcyan";
  };

  function toggleOption(option) {
    if (option == "_clearSelection") {
      value = multi ? [] : null;
      dispatch("change", { value: value });
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
    dispatch("change", { value: value });
  }

  const onKeyDown = (e) => {
    if (!isOpen) {
      return false;
    }
    e.preventDefault();
    if (e.key === "ArrowDown") {
      focusedOptionIdx = Math.min(focusedOptionIdx + 1, options.length - 1);
    } else if (e.key === "ArrowUp") {
      focusedOptionIdx = Math.max(focusedOptionIdx - 1, 0);
    } else if (e.key === "Enter") {
      toggleOption(options[focusedOptionIdx]);
    }
    return true;
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
  class="control"
  bind:this={anchor}
  on:click={(e) => {
    if (inEdit) {
      e.stopPropagation();
      editorState.toggle();
    }
  }}
>
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="inline-value" class:inEdit>
    {#if value.length < 1 && inEdit}
      <span class="placeholder">{placeholder}</span>
    {:else}
      {#each value as val}
        {@const color = optionColors[val] || getOptionColor(val)}
        {#if color}
          <div class="item text" style="--color: {color}">
            <span>
              {val}
            </span>
          </div>
        {:else}
          <div class="text">
            {val || ""}
          </div>
        {/if}
      {/each}
    {/if}
  </div>

  {#if inEdit}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="arrow">
      <Icon name="ChevronDown" />
    </div>
  {/if}

  <Popover on:close={editorState.toggle} {anchor} align={"left"} open={$editorState == "Open"}>
    <div class="options" on:wheel={(e) => e.stopPropagation()}>
      {#if !allowNull}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
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
      {#each options as option, idx}
        {@const color = optionColors[option] || getOptionColor(option)}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
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
    flex: auto;
    display: flex;
    align-items: stretch;
    cursor: pointer;
    overflow: hidden;
  }

  .inline-value {
    flex: auto;
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 0.5rem;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }

  .inline-value .placeholder {
    font-style: italic;
    white-space: nowrap;
    color: var(--spectrum-global-color-gray-600);
  }
  .inEdit {
    background-color: var(
      --spectrum-textfield-m-background-color,
      var(--spectrum-global-color-gray-50)
    );
  }

  .item {
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: var(--color);
    color: white;
    height: 55%;
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
    height: 100%;
    display: grid;
    place-items: center;
    padding-left: 0.3rem;
    padding-right: 0.3rem;
    background-color: var(
      --spectrum-textfield-m-background-color,
      var(--spectrum-global-color-gray-50)
    );
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
