<script>
  import { createEventDispatcher } from 'svelte'
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte"

  export let value
  export let formattedValue
  export let cellState
  export let fieldSchema
  export let unstyled
  export let cellOptions

  const dispatch = createEventDispatcher()

  const handleKeyboard = ( e ) => {
    if ( e.keyCode == 32 ) {
      e.stopPropagation();
      e.preventDefault();
      value = !value
    }
  }

  let anchor

  $: inEdit = $cellState == "Editing"
  $: if ( inEdit && anchor ) anchor?.focus() 
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div 
  class="superCell" 
  class:inEdit={ $cellState == "Editing" }
  class:unstyled
  tabindex="0"
  style:padding-left={cellOptions?.padding}
  style:padding-right={cellOptions?.padding}
  on:keydown={handleKeyboard}
  >
    {#if inEdit }
      <div class="inline-value" > 
        <div class="spectrum-Switch spectrum-Switch--emphasized">
          <input
            bind:this={anchor}
            checked={value}
            on:change={(e) => dispatch("change", { value: e.detail } )}
            type="checkbox"
            class="spectrum-Switch-input"
            on:blur={cellState.lostFocus}
          />
          <span class="spectrum-Switch-switch" />
        </div>
      </div>
    {:else}
      <div class="inline-value" > 
        {#if value}
          <Icon size="S" name="Checkmark" color={"lime"}/>
        {/if}
      </div>
    {/if}
</div>

<style>
  .inline-value { 
    flex: auto;
    display: flex;
    justify-content: var(--super-column-alignment);
    align-items: center;
  }
</style>