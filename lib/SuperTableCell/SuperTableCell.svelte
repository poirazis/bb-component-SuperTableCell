<script>
  import { getContext , createEventDispatcher } from "svelte";

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
  $: console.log(cellState ? $cellState : "No State Yet") 
</script>

<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div 
  class="superTableCellWrapper"
  style:color={cellOptions?.color}
  class:inEdit={$cellState == "Editing"}
  tabindex="-1"
  use:clickOutsideAction
  on:keydown={handleKeyboard}
  on:clickoutside={() => { 
    if ($cellState != "View") {
      console.log("Clicked Outside")
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

:global(.superTableCellWrapper.inEdit) {
  width: var(--lock-width);
  max-width: var(--lock-width);
  color: var(--spectrum-global-color-gray-900);
  border-color: var(--spectrum-global-color-gray-600);
}
:global(.superTableCellWrapper.inEdit::before) {
  content: "";
  position: absolute;
  top: 1;
  right: 1;
  left: 1;
  bottom: 1;
  filter: brightness(80%);
  background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
}
:global(.superTableCell.focused) {
    border-color: var(--spectrum-global-color-gray-500);
  }
:global(.superTableCell:focus) {
    outline: none;
  }
:global(.overflow) {
    position: absolute;
    right: 0;
    top: 20%;
    height: 60%;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    width: calc( 4 * var(--super-table-cell-padding));
    mask-image: linear-gradient(to right, transparent 0%, black 75% );
    mask-mode: alpha;
    z-index: 1;
    transition: all 130ms;
    border: none;
  }
:global(.overflow.inEdit) {
    width: calc( 6 * var(--super-table-cell-padding));
    mask-image: linear-gradient(to right, transparent 0%, black 60% );
  }
</style>