<script>
  import { getContext , onMount } from 'svelte'
  import CellGeneric from './lib/CellGeneric.svelte';
  import CellRelationship from './lib/CellRelationship.svelte';

  const { styleable, builderStore  } = getContext("sdk")
  const component = getContext("component")
  const columnInfo = getContext("SuperTableColumnInfo")
  const tableStore = getContext("tableStore")

  export let value, rowKey, required = true
  export let valueColor
  export let editable
  export let cellType
  export let viewType

  let autobind = true

  function pickyBinder()  {
    // Auto bind the value to the Super Column CellValue binding if plaed in one
    if ( $builderStore?.inBuilder ) {
      let binding = "{{ [" + columnInfo?.parentID + "].[cellValue]" + " }}"
      builderStore?.actions.updateProp("value", binding)
      value = binding
      autobind = false
    }
  }
  // Append Styles Super Table Styling variables
  $: styles = {
    ...$component?.styles,
    normal: {
      ...$component?.styles.normal
    },
  };

  onMount( () => { if (columnInfo && !value && autobind) pickyBinder() } )
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="super-table-cell" use:styleable={styles}>
  <!-- Check valid placement  -->
  {#if !columnInfo || !tableStore}
    <p> Super Table Cells need to be placed inside Super Table Column Component </p>
  {:else}
    {value}
  {/if}
</div>

<style>
  .super-table-cell {
    flex: 1;
    display: block;
    max-width: 350px !important;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

</style>