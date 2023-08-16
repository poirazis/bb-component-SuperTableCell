<script>
  import { createEventDispatcher } from 'svelte'
  import DatePicker from "../../../node_modules/@budibase/bbui/src/Form/Core/DatePicker.svelte"

  export let value
  export let inEdit = false
  export let formattedValue

  const dispatch = createEventDispatcher()

</script>

<div class="control" class:inEdit>
  {#if inEdit}
    <DatePicker {value} on:change={(e) => dispatch("change", {value : e.detail } ) } />
  {:else}
    <div class="inline-value"> 
      { formattedValue || value || "" }
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
    flex-grow: 1;
    display: flex;
    align-items: center;
  }

 .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
  .inline-value { 
    flex-grow: 1;
    display: flex;
    white-space: nowrap;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
    justify-content: var(--super-column-alignment);
    align-items: center;
  }
</style>