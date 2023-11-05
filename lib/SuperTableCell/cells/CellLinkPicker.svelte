<script>
    import CellSkeleton from "./CellSkeleton.svelte";
    import { getContext , createEventDispatcher } from "svelte";
    import { fly } from "svelte/transition"

    const { API } = getContext("sdk");
    const dispatch = createEventDispatcher();

    export let value = []
    export let tableId 
    export let schema
    export let active = false
    
    let timer;
    let limit = 10

    let tableSchema
    let filteredValue = ""
    let searchColumns = []
    let results
    let queryParams = {}
    let primaryDisplay

    const debounce = e => {
      clearTimeout(timer);
      timer = setTimeout(() => {
        filteredValue = e.target.value;
      }, 500);
    } 

    const setLimit = (e, val ) => {
      e.stopPropagation();
      limit = val
    }

    const loadTableSchema = async ( tableId ) => {
      if ( tableId && !tableSchema ) {
        tableSchema = await API.fetchTableDefinition(tableId);
        searchColumns = [ tableSchema.primaryDisplay ]
        primaryDisplay = tableSchema.primaryDisplay
      }
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

    const rowSelected = ( val ) => {
      if ( value ) {
        let found = value.find ( (e) => e._id == val._id )
        return found 
      }
    }

    const selectRow = ( val ) => {

      if ( schema.relationshipType == "many-to-many" ) {
        value.push ( { _id: val._id, primaryDisplay: val[tableSchema.primaryDisplay] } )
      } else if ( schema.relationshipType == "many-to-one") {
        value.push ( { _id: val._id, primaryDisplay: val[tableSchema.primaryDisplay] } )
      } else if ( schema.relationshipType == "one-to-many") {
        value = [ { _id: val._id, primaryDisplay: val[tableSchema.primaryDisplay] }]
      }
      dispatch("change", value )
    }

    const unselectRow = ( val ) => {
      value.splice( value.findIndex ( (e) => e._id === val._id  ), 1 );
      dispatch("change", value )
    }

    $: loadTableSchema(tableId) 
    $: results = loadTable(tableId, filteredValue, limit )

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="control" style:min-width={schema.relationshipType == "one-to-many" ? " 160px" : "320px"}> 

  <div class="searchControl">
    <div class="columnSelect">
      {primaryDisplay} - {schema.relationshipType}
    </div>
    <input class="input" on:input={debounce} type="text" placeholder="Search..."/>
    <div class="pageSize">
      <div class="pageSizeItem" class:selected={limit == 10} on:click={(e) => setLimit(e, 10)}> 10 </div>
      <div class="pageSizeItem" class:selected={limit == 50} on:click={(e) => setLimit(e, 50)}> 50 </div>
      <div class="pageSizeItem" class:selected={limit == 100} on:click={(e) => setLimit(e, 100)}> 100 </div>
      <div class="pageSizeItem" class:selected={limit == 10000} on:click={(e) => setLimit(e, 10000)}> ALL </div>
    </div>
  </div>

  {#if schema.relationshipType == "many-to-many" || schema.relationshipType == "many-to-one"}
    <div class="listWrapper">
      <div class="list">
        <div class="options"> 
          {#if results}
            {#await results}
              <CellSkeleton > <div class="option text"> Loading ... </div> </CellSkeleton> 
            {:then results}
              {#key value}
                {#if results.rows.length > 0 }
                  {#each results.rows as row, idx }
                    {#if !(rowSelected(row)) }
                      <div class="option" on:click={selectRow(row)} >
                        <div class="option text">
                          {row[primaryDisplay]}
                        </div>
                      </div>
                    {/if}
                  {/each}
                {/if}
              {/key}
            {:catch error}
              <p style="color: red">{error.message}</p>
            {/await}
          {/if}
        </div>
      </div>
      <div class="list listSelected">
        <div class="options"> 
          {#each value as val, idx }
            {#if (rowSelected(val)) }
              <div transition:fly={{ x: -20, duration: 130}} class="option" on:click={unselectRow(val)} >
                <div class="option text">
                  {val.primaryDisplay}
                </div>
              </div>
            {/if}
          {/each}
        </div>
      </div>
    </div>
  {/if}

  {#if schema.relationshipType == "one-to-many"}
    <div class="listWrapper">
      <div class="list" style:width={"100%"}>
        <div class="options"> 
          {#if results}
            {#await results}
              <CellSkeleton > <div class="option text"> Loading ... </div> </CellSkeleton> 
            {:then results}
              {#if results.rows.length > 0 }
                {#key value}
                  {#each results.rows as row, idx }
                    {#if !(rowSelected(row)) }
                      <div class="option" on:click={selectRow(row)} >
                        <div class="option text">
                          {row[primaryDisplay]}
                        </div>
                      </div>
                    {/if}
                  {/each}
                {/key}
              {/if}
            {:catch error}
              <p style="color: red">{error.message}</p>
            {/await}
          {/if}
        </div>
      </div>
    </div>
  {/if}

</div>


<style>
   .control {
    flex: auto;
    flex-direction: column;
    display: flex;
    align-items: stretch;
    justify-content: space-around;
    gap: 0.85rem;
    padding: 0.5rem;
  }

  .searchControl {
    flex: 0 0 auto;
  }

  .listWrapper {
    flex: auto;
    display: flex;
    justify-content: stretch;
    align-content: stretch;
    gap: 0.85rem;
  }

  .list {
    width: 50%;
    height: 200px;
    overflow-y: auto;
    overflow-x: hidden;
    color: var(--spectrum-global-color-gray-700);
    border: 1px solid var(--spectrum-global-color-gray-300);
    border-radius: 0.25rem;
  }

  .listSelected {
    color: var(--spectrum-global-color-gray-900);
    border: 1px solid var(--spectrum-global-color-gray-500);
  }
  .searchControl {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    min-height: 2rem;
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
    color: var(--spectrum-global-color-gray-600);
    display: flex;
    justify-content: center;
    border: 1px solid var(--spectrum-global-color-gray-300);
    border-radius: 4px;
    cursor: pointer;
  }

  .pageSizeItem.selected {
    color: var(--primaryColor);
    background-color: var(--spectrum-global-color-gray-300);
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
    gap: 0.3rem;
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
</style>