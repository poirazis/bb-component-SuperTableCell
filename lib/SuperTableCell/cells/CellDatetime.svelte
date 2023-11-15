<script>
  import { createEventDispatcher } from 'svelte'
  import DatePicker from "../../../node_modules/@budibase/bbui/src/Form/Core/DatePicker.svelte"

  export let value
  export let cellState
  export let formattedValue
  export let unstyled
  export let cellOptions

  const dispatch = createEventDispatcher()

  let anchor 
  let pickerStatus = "Closed"

  $: inEdit = $cellState == "Editing"
  $: if ( inEdit && anchor ) anchor.focus();
</script>

<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  bind:this={anchor}
  class="superCell"
  class:unstyled 
  class:inEdit
  tabindex="0"
  style:padding-left={cellOptions?.padding}
  style:padding-right={cellOptions?.padding}
  on:blur={ () => pickerStatus == "Closed" ? cellState.lostFocus() : null }
>
  {#if inEdit}
    <div class="pickerWrapper">
      <DatePicker 
        {value} 
        on:open={ (e) => pickerStatus = "Open" }
        on:close={ (e) => { pickerStatus = "Closed"; } }
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