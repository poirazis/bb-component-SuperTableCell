<script>
  import { getContext , beforeUpdate, createEventDispatcher } from "svelte";

  import { clickOutsideAction } from "svelte-legos";
  import SuperCell from "./SuperCell.svelte";

  const tableDataChangesStore = getContext("tableDataChangesStore");
  const dispatch = createEventDispatcher();

  export let value
  export let rowKey
  export let editable
  export let fieldSchema
  export let valueTemplate
  export let submitOn = "onEnter"
  export let isHovered = false

  export let cellOptions

  let originalValue = Array.isArray(value) ? [ ... value ] : value

  let cellState
  let width

  function acceptChange ( ) { 
    let newDataChange = {
      rowID: rowKey,
      field: fieldSchema.name,
      originalValue: originalValue,
      newValue: value
    };

    // Get all other dataChanges, removing any previous changes of ours
    let dataChanges = $tableDataChangesStore.filter(
      (v) => v.rowKey != rowKey || v.field != fieldSchema.name
    );
    originalValue = Array.isArray(value) ? [ ... value ] : value
    $tableDataChangesStore = [...dataChanges, newDataChange];
  }


  function handleChange ( e ) {
    value = e.detail
    console.log("Change Received", value )
  }

  const handleKeyboard = ( e ) => {
    if (e.key == "Escape" ) {
      e.stopPropagation();
      e.preventDefault();
      cellState.unfocus();
    }
  }

  let wrapperAnchor
  let focusedOrHasFocused

  $: console.log(cellState ? $cellState : "No State Yet") 

  // We need to know if our child Super Cell has the focus
  beforeUpdate( () => {
    focusedOrHasFocused = wrapperAnchor?.matches(":focus-within") ?? false
  })

</script>

<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div 
  bind:this={wrapperAnchor}
  class="superTableCellWrapper"
  style:color={cellOptions?.color}
  class:inEdit={$cellState == "Editing" }
  class:focused={focusedOrHasFocused}
  tabindex="-1"
  use:clickOutsideAction
  on:keydown={handleKeyboard}
  on:clickoutside={() => { 
    if ( $cellState != "View") {
      cellState.lostFocus()
  }}}
  on:click={cellState.focus}
>
  <SuperCell
    bind:cellState 
    {value}
    {valueTemplate}
    {fieldSchema}
    {editable}
    {isHovered}
    unstyled
    on:change={handleChange}
  />
</div>

<style>
.superTableCellWrapper {
  flex: auto;
  display: flex;
  align-items: stretch;
  justify-content: stretch;
  overflow: hidden;
  padding-left: var(--super-table-cell-padding);
  padding-right: var(--super-table-cell-padding);
  border: 1px solid transparent;
  min-width: 0;
}

.superTableCellWrapper.inEdit {
  width: var(--lock-width);
  max-width: var(--lock-width);
  color: var(--spectrum-global-color-gray-900);
  border-color: var(--spectrum-global-color-gray-600);
}
.superTableCellWrapper.inEdit::before {
  content: "";
  position: absolute;
  top: 1;
  right: 1;
  left: 1;
  bottom: 1;
  filter: brightness(80%);
  background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
}
.superTableCellWrapper.focused {
    border-color: var(--spectrum-global-color-gray-500);
  }
.superTableCellWrapper:focus-within {
    border-color: var(--spectrum-global-color-blue-500);
  }
.superTableCellWrapper:focus {
    outline: none;
    border-color: var(--spectrum-global-color-red-500);
  }
</style>