<script>
  import { createEventDispatcher } from 'svelte'

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
        <svg xmlns="http://www.w3.org/2000/svg" 
          width="14" height="14" viewBox="0 0 24 24" 
          fill="none" stroke="lime" 
          stroke-width="2" 
          stroke-linecap="round" 
          stroke-linejoin="round" 
          class="lucide lucide-check">
            <path d="M20 6 9 17l-5-5"/>
          </svg>
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