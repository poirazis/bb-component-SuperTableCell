<script>
  import { getContext } from "svelte";
  import fsm from "svelte-fsm";
  import CellString from "./cells/CellString.svelte";
  import CellArray from "./cells/CellArray.svelte";
  import CellLink from "./cells/CellLink.svelte";
  import CellDatetime from "./cells/CellDatetime.svelte";
  import CellBoolean from "./cells/CellBoolean.svelte"
  import CellAttachment from "./cells/CellAttachment.svelte";

  const columnContext = getContext("columnContext");
  const tableDataChangesStore = getContext("tableDataChangesStore");

  export let value,
    rowKey,
    required = true;
  export let valueColor;
  export let editable;
  export let columnType;
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
  {#if columnType === "string"}
    <CellString
      on:enterEdit={cellState.edit}
      on:submit={cellState.submit}
      on:cancelEdit={cellState.cancel}
      {editable}
      {value}
    />
  {:else if columnType === "datetime"}
    <CellDatetime
      on:enterEdit={cellState.edit}
      on:submit={cellState.submit}
      on:cancelEdit={cellState.cancel}
      {editable}
      {value}
    />
  {:else if columnType === "link"}
    <CellLink
      on:enterEdit={cellState.edit}
      on:submit={cellState.submit}
      on:cancelEdit={cellState.cancel}
      editable={false}
      {value}
    />
  {:else if columnType === "boolean"}
    <CellBoolean
      on:enterEdit={cellState.edit}
      on:submit={cellState.submit}
      on:cancelEdit={cellState.cancel}
      {editable}
      {value}
    />
  {:else if columnType === "array"}
    <CellArray
      on:enterEdit={cellState.edit}
      on:submit={cellState.submit}
      on:cancelEdit={cellState.cancel}
      {editable}
      {value}
    />
  {:else if columnType === "attachment"}
    <CellAttachment
      on:enterEdit={cellState.edit}
      on:submit={cellState.submit}
      on:cancelEdit={cellState.cancel}
      {editable}
      {value}
    />
  {/if}
</div>

<style>
  .superTableCell {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 100%;
    height: 100%;
    background: transparent;
    color: var(--super-column-color);
  }

  .inEdit {
    border-width: 1px;
    border-style: solid;
    border-color: var(--spectrum-alias-border-color-mouse-focus);
  }
</style>