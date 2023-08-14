<script>
  import { createEventDispatcher } from 'svelte'
  import DatePicker from "../../../node_modules/@budibase/bbui/src/Form/Core/DatePicker.svelte"

  export let value
  export let inEdit = false

  const dispatch = createEventDispatcher()
  let editing = false, original


  $: fmtDate = new Date(value)
</script>

<div class="control" class:inEdit>
  {#if inEdit}
    <DatePicker {value} on:change={(e) => dispatch("change", {value : e.detail } ) } />
  {:else}
    <div class="inline-value"> 
      {fmtDate.toLocaleDateString('en-us', { day:"numeric", year:"numeric", month:"long"}) } 
    </div>
  {/if}
</div>

<style>

  :global(.control > .spectrum-Datepicker > .spectrum-Textfield > .spectrum-Textfield-input ) {
    border: none;
  }
  :global(.control > .spectrum-Datepicker > .spectrum-Picker ) {
    padding: 0.2rem;
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
    border: none;
  }
  .control {
    min-height: 2rem;
    display: flex;
    align-items: stretch;
  }

 .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
  .inline-value { 
    flex: auto;
    display: flex;
    flex-direction: column;
    justify-content: center;
    white-space: nowrap;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }
</style>