<script>
  import { createEventDispatcher } from 'svelte'
  import DatePicker from "../../../node_modules/@budibase/bbui/src/Form/Core/DatePicker.svelte"

  export let value
  export let cellState
  export let formattedValue
  export let unstyled
  export let width
  export let cellOptions

  const dispatch = createEventDispatcher()

  $: inEdit = $cellState == "Editing" || $cellState == "EditingWithEditor"

</script>

<div 
  class="superCell"
  class:unstyled 
  class:inEdit
  style:padding-left={cellOptions?.padding}
  style:padding-right={cellOptions?.padding}
  style:max-width={width} 
>
  {#if inEdit}
    <div class="pickerWrapper">
      <DatePicker 
        {value} 
        on:open={cellState.openEditor}
        on:close={cellState.closeEditor}
        on:change={(e) => dispatch("change", e.detail ) } />
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
    background-color: unset;
    border: unset;
    padding: unset;
  }
  :global(.pickerWrapper > .spectrum-Datepicker > .spectrum-Textfield ) {
    background-color: unset;
    border: unset;
    padding: unset;
  }
  :global(.pickerWrapper > .spectrum-Datepicker > .spectrum-Picker ) {
    background: unset;
    border: unset;
    padding: unset;
  }
  .inline-value { 
    flex-grow: 1;
    display: flex;
    white-space: nowrap;
    justify-content: var(--super-column-alignment);
    align-items: center;
  }

</style>