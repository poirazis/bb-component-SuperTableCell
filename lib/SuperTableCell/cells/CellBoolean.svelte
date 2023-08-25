<script>
  import { createEventDispatcher, onMount } from 'svelte'
  import Switch from "../../../node_modules/@budibase/bbui/src/Form/Core/Switch.svelte"
  import Icon from "../../../node_modules/@budibase/bbui/src/Icon/Icon.svelte"

  export let value
  export let inEdit

  const dispatch = createEventDispatcher()

  $: dispatch("change", { value : value} )

</script>

<div class="control" class:inEdit> 
  {#if !inEdit}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="inline-value" > 
    {#if value}
      <Icon size="XS" name="Checkmark" color={"lime"}/>
    {/if}
  </div>
  {:else}
    <div class="inline-value" > 
      <Switch {value} on:change={(e) => dispatch("change", { value: e.detail } )}/>
    </div>
  {/if}
</div>

<style>
  .inline-value { 
    flex: auto;
    display: flex;
    justify-content: var(--super-column-alignment);
    align-items: center;
    padding-left: var(--super-table-cell-padding);
    padding-right: var(--super-table-cell-padding);
  }

  .control {
    flex: auto;
    display: flex;
    align-items: stretch;
    justify-content: stretch;
  }

  .inEdit {
    background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  }
</style>