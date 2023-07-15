<script>
  import { getContext, onMount } from "svelte";
  import fsm from "svelte-fsm";
  import CellString from "./cells/CellString.svelte";

  const columnContext = getContext("columnContext");
  const tableDataChangesStore = getContext("tableDataChangesStore");

  export let cellOptions 
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
  const cellState = fsm("entering", {
    idle: { edit: "entering " },

    entering: {
      submit: "submitting",
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

  // Event Handlers
  function handleEnterEdit() {
    console.log("Editing");
  }

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

  function handleCancelEdit() {
    console.log("Editing Cancelled");
  }
</script>

<CellString
  on:enterEdit={handleEnterEdit}
  on:submit={handleSubmit}
  on:cancelEdit={handleCancelEdit}
  {value}
  {cellOptions}
/>
