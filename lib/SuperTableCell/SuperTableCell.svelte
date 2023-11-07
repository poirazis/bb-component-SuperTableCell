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
  export let columnOptions

  let originalValue = Array.isArray(value) ? [ ... value ] : value

  let cellState
  let width

  const getCellColor = (value, template) => {
    if (!template) {
      return null
    }
    return processStringSync(template,  { Value: value }  )
  }

  /** @type {import('./SuperCell.Svelte').cellOptions} */
  $: cellOptions = {
    align: columnOptions.align,
    color: columnOptions.color,
    background: columnOptions.background ?? "transparent",
    fontWeight: columnOptions.fontWeight,
    padding: columnOptions.padding
  }

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
  style:--lock-width={width}
  class:inEdit={$cellState == "Editing" }
  class:focused={focusedOrHasFocused}
  tabindex="0"
  use:clickOutsideAction
  on:keydown={handleKeyboard}
  on:clickoutside={() => { 
    if ( cellState && $cellState != "View") {
      cellState.lostFocus()
  }}}
  on:click={() => { width = wrapperAnchor.clietWidth ; cellState.focus() }}
  on:focus={() => cellState.focus() }
>
  <SuperCell
    bind:cellState 
    {cellOptions}
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
.superTableCellWrapper.inEdit.focused {
    border-color: var(--spectrum-global-color-gray-500);
  }
.superTableCellWrapper.inEdit:focus-within {
    border-color: var(--spectrum-global-color-blue-500);
  }
</style>