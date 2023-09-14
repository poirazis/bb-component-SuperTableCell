<script>
  import { createEventDispatcher } from 'svelte'
  import DatePicker from "../../../node_modules/@budibase/bbui/src/Form/Core/DatePicker.svelte"

  export let value
  export let cellState
  export let formattedValue

  const dispatch = createEventDispatcher()

  $: inEdit = $cellState == "Editing"

</script>

<div class="superCell" class:inEdit>
  {#if inEdit}
    <div class="pickerWrapper">
      <DatePicker {value} on:change={(e) => dispatch("change", {value : e.detail } ) } />
    </div>
  {:else}
    <div class="inline-value"> 
      { formattedValue || value || "" }
    </div>
  {/if}
</div>

<style>

  .pickerWrapper {
    flex: auto;
    display: flex;
    align-items: center;
  }

  :global(.pickerWrapper > .spectrum-Datepicker > .spectrum-Textfield > .spectrum-Textfield-input ) {
    border: unset;
    padding: unset;
  }
  :global(.pickerWrapper > .spectrum-Datepicker > .spectrum-Picker ) {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
    border: unset;
    padding: unset;
  }


 .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
  .inline-value { 
    flex-grow: 1;
    display: flex;
    white-space: nowrap;
    justify-content: var(--super-column-alignment);
    align-items: center;
  }
</style>