<script>
  import { getContext } from "svelte";
  import fsm from "svelte-fsm";
  import CellString from "./cells/CellString.svelte";

  const columnContext = getContext("columnContext");
  const tableDataChangesStore = getContext("tableDataChangesStore");

  export let value,
    rowKey,
    required = true;
  export let valueColor;
  export let editable;
  export let cellType;
  export let viewType;

  export let fontColor, fontSize, isBold, isUnderline, isItalic;

  let originalValue;

  // Setup Cell State Machine
  const cellState = fsm("idle", {
    idle: { edit: "entering" },

    entering: {
      submit: "submitting",
      cancel: "idle"
    },

    submitting: {
      _enter() {
        const body = new URLSearchParams({ value });
        fetch("https://some.endpoint", { method: "POST", body })
          .then(this.success)
          .catch(this.error);
      },

      success: "idle",

      error(err) {
        error = err;
        return "invalid";
      },
    },

    invalid: {
      input: "entering",
    },

    completed: {},
  });

  function handleSubmit(event) {
    let newDataChange = {
      rowKey: rowKey,
      field: columnContext?.columnField,
      originalValue: originalValue,
      newValue: event.detail.newValue,
    };

    // Get all other dataChanges, removing any previous changes of ours
    let dataChanges = $tableDataChangesStore.filter(
      (v) => v.rowKey != rowKey || v.field != columnContext?.columnField
    );

    $tableDataChangesStore = [...dataChanges, newDataChange];
  }

</script>

<!-- Will Reder Different CellTypes depending on the column type or manual override -->
<div 
  class="superTableCell"
  class:inEdit={$cellState === "entering"}
>
  <CellString
    on:enterEdit={cellState.edit}
    on:submit={cellState.submit}
    on:cancelEdit={cellState.cancel}
    {editable}
    {value}
  />
</div>

<style>
  .superTableCell {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 100%;
    height: 100%;
  }

  .inEdit {
    border-width: 1px;
    border-style: solid;
    border-color: var(--spectrum-alias-border-color-mouse-focus);
  }
</style>