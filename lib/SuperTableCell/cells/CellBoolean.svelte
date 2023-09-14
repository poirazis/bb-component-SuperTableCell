<script>
  import { createEventDispatcher, onMount } from 'svelte'
  import Switch from "../../../node_modules/@budibase/bbui/src/Form/Core/Switch.svelte"
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte"

  export let value
  export let formattedValue
  export let cellState
  export let fieldSchema
  export let unstyled

  const dispatch = createEventDispatcher()

  $: dispatch("change", { value : value} )

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div 
  class="superCell" 
  class:inEdit={ $cellState == "Editing" }
  class:unstyled
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

  .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>