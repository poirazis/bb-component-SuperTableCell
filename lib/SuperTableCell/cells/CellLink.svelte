<script>
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte";
  import Popover from "../../../node_modules/@budibase/bbui/src/Popover/Popover.svelte";
  import fsm from "svelte-fsm";
  import { getContext, createEventDispatcher } from "svelte";
  import CellSkeleton from "./CellSkeleton.svelte";

  const dispatch = createEventDispatcher();
  const { API } = getContext("sdk");

  export let value;
  export let inEdit;
  export let formattedValue;
  export let fieldSchema;

  let anchor;
  let results
  let allowNull = fieldSchema.constraints.presence ?? false;
  let relTableSchema 
  let relTableSeachColumn
  let filteredValue

  let editorState = fsm("Closed", {
    Open: { toggle: "Closed" },
    Closed: { toggle: "Open" },
  });

  const loadTable = async () => {
    results = await API.searchTable({
      paginate: false,
      tableId: fieldSchema.tableId,
      limit: 50,
      query: { fuzzy: { "Customer Name": filteredValue } }
    });
    return results;
  };

  const loadTableSchema = async () => {
    relTableSchema = await API.fetchTableDefinition(fieldSchema.tableId);
    return relTableSchema;
  }

  const rowSelected = ( val ) => {
    let found = value.find ( (e) => e._id == val._id )
    return found
  }

  const selectRow = ( val ) => {
    value.push ( { _id: val._id, primaryDisplay: val[relTableSchema.primaryDisplay] } )
    value = value
  }

  const unselectRow = ( e, val ) => {
    e.stopPropagation();
    value.splice( value.findIndex ( (e) => e._id === val._id  ), 1 );
    value = value;
  }

  const getPreviousColumn = ( e , val ) => {
    e.stopPropagation();
    relTableSeachColumn = "Firt Name"
  }

  const getNextColumn = ( e , val ) => {
    e.stopPropagation();
    relTableSeachColumn = "Surname"
  }

  function openPicker(e) {
    e.stopPropagation();
    editorState.toggle();
    if ( $editorState == "Open" ) {
      if (!relTableSchema) relTableSchema = loadTableSchema();
      if (!results || filteredValue ) results = loadTable();
    }
  }

  $: results = loadTable(filteredValue);

  $: console.log(relTableSeachColumn)
</script>

<div bind:this={anchor} class="control">
  {#if inEdit}
    <div class="inline-value">
      {#if value}
        {#each value as row, idx}
          <div class="item">
            {row.primaryDisplay}
            <Icon size="XS" name="Close" hoverable on:click={ (e) => unselectRow(e, row)}/>
          </div>
        {/each}
      {/if}
      <Icon name="Add" hoverable size="M" on:click={(e) => openPicker(e)} />
    </div>

    <Popover
      on:close={editorState.toggle}
      {anchor}
      useAnchorWidth
      open={$editorState == "Open"}
    >
      <div class="searchControl">
        <div class="columnSelect">
          <Icon hoverable name="ChevronLeft" size="S" on:click={(e) => getPreviousColumn(e)} />
          {relTableSeachColumn}
          <Icon hoverable name="ChevronRight" size="S" on:click={(e) => getNextColumn(e)} />
        </div>
        <input class="input" bind:value={filteredValue} type="text" />
      </div>
      <div class="options"> 
        {#await results}
          <CellSkeleton > Loading... </CellSkeleton>
        {:then results}
        {#key value}
          {#if results.rows.length > 0 }
            {#each results.rows as row, idx }
              {#if !(rowSelected(row)) }
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <div class="option" on:click={selectRow(row)}
                >
                { row[relTableSchema.primaryDisplay] } 
                </div>
              {/if}
            {/each}
          {/if}
          {/key}
        {:catch error}
          <p style="color: red">{error.message}</p>
        {/await}
      </div>
    </Popover>
  {:else}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="inline-value">
      {#if value}
        {#each value as row}
          <div class="item">{row.primaryDisplay}</div>
        {/each}
      {/if}
    </div>
  {/if}
</div>

<style>
  .control {
    flex: auto;
    display: flex;
    align-items: stretch;
    justify-content: stretch;
  }
  .inline-value {
    flex: auto;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    column-gap: 0.5rem;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }

  .searchControl {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    min-height: 2rem;
    width: 100%;
    padding: 0.5rem;
  }

  .columnSelect {
    flex: auto;
    display: flex;
    justify-content: space-between;
    align-items: center;
    min-height: 2rem;
    font-weight: 800;
  }

  .input {
    min-height: 2rem;
  }
  .options {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    padding: 0.5rem;
  }

  .option {
    flex: auto;
    display: flex;
    justify-content: flex-start;
    cursor: pointer;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
  }
  .option:hover {
    background-color: var(--spectrum-global-color-gray-100);
  }
  .item {
    flex: 0 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: #407c36;
    color: whitesmoke;
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    white-space: nowrap;
    gap: 0.25rem;

  }
</style>
