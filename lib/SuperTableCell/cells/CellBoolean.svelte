<script>
  import { createEventDispatcher } from 'svelte'
  import Switch from "../../../node_modules/@budibase/bbui/src/Form/Core/Switch.svelte"
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

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div 
  class="superCell" 
  class:inEdit={ $cellState == "Editing" }
  class:unstyled
  tabindex="-1"
  style:padding-left={cellOptions?.padding}
  style:padding-right={cellOptions?.padding}
  on:keydown={handleKeyboard}
  >
    {#if $cellState == "Editing" }
      <div class="inline-value" > 
        <Switch {value} on:change={(e) => dispatch("change", { value: e.detail } )}/>
      </div>
    {:else}
    <div class="inline-value" > 
      {#if value}
        <Icon size="XS" name="Checkmark" color={"lime"}/>
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