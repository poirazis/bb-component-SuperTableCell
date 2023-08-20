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
  let relTableSchema 
  let relTableSeachColumn
  let filteredValue
  let searchColumns 
  let primaryDisplay 
  let queryParams = {}
  let limit = 10

  let editorState = fsm("Closed", {
    Open: { toggle: "Closed" },
    Closed: { toggle: "Open" },
  });

  let timer;
	const debounce = e => {
    clearTimeout(timer);
		timer = setTimeout(() => {
      filteredValue = e.target.value;
		}, 500);
	} 

  const loadTable = async ( tableId, filterValue, limit ) => {

    queryParams = {}
    queryParams[primaryDisplay] = filterValue ?? ""

    results = API.searchTable({
      paginate: false,
      tableId: tableId,
      limit: Number (limit),
      query: { fuzzy: queryParams }
    });
    return results;
  };

  const loadTableSchema = async () => {
    relTableSchema = await API.fetchTableDefinition(fieldSchema.tableId);
    searchColumns = [ relTableSchema.primaryDisplay ]
    primaryDisplay = relTableSchema.primaryDisplay
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
    searchColumns.push ( "Customer Email" );
    searchColumns = searchColumns;
  }

  const setLimit = (e, val ) => {
    e.stopPropagation();
    limit = val
  }

  function openPicker(e) {
    e.stopPropagation();
    editorState.toggle();
    if ( $editorState == "Open" ) {
      
      if (!relTableSchema) relTableSchema = loadTableSchema();
      if (!results || filteredValue ) results = loadTable();
    }
  }

  function focus(element) {
		element.focus()
	}

  $: if ( inEdit && !primaryDisplay ) loadTableSchema();
  $: if ( $editorState == "Open" ) loadTable( fieldSchema.tableId, filteredValue , limit)
  $: if ( !inEdit && $editorState == "Open" ) { editorState.toggle(); filteredValue = ""; }

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div bind:this={anchor} class="control">

  {#if inEdit}
    <div class="inline-value">
      {#if value}
        {#each value as row }
          <div class="item">
            {row.primaryDisplay}
            <Icon size="XS" name="Unlink" hoverable color={"var(--spectrum-global-color-gray-50)"} on:click={ (e) => unselectRow(e, row)}/>
          </div>
        {/each}
      {/if}
      <Icon name="Add" hoverable size="M" on:click={(e) => openPicker(e)} />
    </div>

    <Popover
      on:close={editorState.toggle}
      {anchor} 
      align = "left"
      open={$editorState == "Open"}
    >
      <div class="columnWrapper"> 
      {#each searchColumns as searchColumn}
      <div class="column">
        <div class="searchControl">
          <div class="columnSelect">
            {searchColumn}
          </div>
          <input class="input" on:input={debounce} type="text" use:focus/>
          <div class="pageSize">
            <div class="pageSizeItem" class:selected={limit == 10} on:click={(e) => setLimit(e, 10)}> 10 </div>
            <div class="pageSizeItem" class:selected={limit == 50} on:click={(e) => setLimit(e, 50)}> 50 </div>
            <div class="pageSizeItem" class:selected={limit == 100} on:click={(e) => setLimit(e, 100)}> 100 </div>
            <div class="pageSizeItem" class:selected={limit == 10000} on:click={(e) => setLimit(e, 10000)}> ALL </div>
          </div>
        </div>
        <div class="options"> 
          {#await results}
            <CellSkeleton > Loading... </CellSkeleton>
            <CellSkeleton > Loading... </CellSkeleton>
            <CellSkeleton > Loading... </CellSkeleton>
          {:then results}
            {#key value}
              {#if results.rows.length > 0 }
                {#each results.rows as row, idx }
                  {#if !(rowSelected(row)) }
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <div class="option" on:click={selectRow(row)} >
                      <Icon name="Link" size="S" color={"var(--primaryColor)"} />
                      <div class="option text">
                         {row[searchColumn]}
                      </div>
                    </div>
                  {/if}
                {/each}
              {/if}
            {/key}
          {:catch error}
            <p style="color: red">{error.message}</p>
          {/await}
        </div>
      </div>
      {/each}
    </div>
    </Popover>
  {:else}
    <div class="inline-value">
      {#if value}
        {#each value as row}
          <div class="item"> {row.primaryDisplay} </div>
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

  .columnWrapper {
    flex: auto;
    display: flex;
    align-items: stretch;
    gap: 1rem;    
  }
  .column {
    flex: auto;
    display: flex;
    flex-direction: column;
    align-items: stretch;
  }

  .searchControl {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    min-height: 2rem;
    padding: 0 0.5rem;
  }

  .columnSelect {
    flex: auto;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-weight: 800;
    min-height: 1.85rem;
  }

  .input {
    min-height: 1.85rem;
    min-width: none;
    width: 100%;
    box-sizing: border-box;
    outline: none;
    background: none;
    border: 1px solid var(--primaryColor);
    padding-left: 0.5rem;
    color: inherit;
    font: inherit;
    cursor: pointer;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }

  .pageSize {
    flex: auto;
    display: flex;
    justify-content: space-between;
    padding-top: 0.5rem;
  }
  .pageSizeItem {
    width: 24%;
    color: var(--primaryColor);
    display: flex;
    justify-content: center;
    border: 1px solid var(--primaryColor);
    border-radius: 4px;
    cursor: pointer;
  }

  .pageSizeItem.selected {
    color: white;
    background-color: var(--primaryColor);
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
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    cursor: pointer;
  }

 :global(.option > span) {
    color: lime;
  }
  .option:hover,
  .option.focused {
    background-color: var(--spectrum-global-color-gray-200);
    border-radius: 4px;
  }

  .text {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .item {
    flex: 0 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    background-color: darkcyan;
    color: whitesmoke;
    height: 60%;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    white-space: nowrap;
    gap: 0.5rem;

  }
</style>
