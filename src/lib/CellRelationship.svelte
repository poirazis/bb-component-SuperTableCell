<script>
    import { getContext } from "svelte"; 
    export let value
		export let open 
		export let viewType
		export let fontSize = "M"

		let _value
    $: {
        try {
            _value = JSON.parse(value)
        } catch {
            _value = null
        }
    }

		function handleClick () {
			open = !open;
		}
</script>

	{#if _value}

		{#if viewType == "tree"}

			<ul class="spectrum-TreeView spectrum-TreeView--sizeS" style="width: 100%">
					<!-- svelte-ignore a11y-click-events-have-key-events -->
					<li on:click={handleClick} class="spectrum-TreeView-item" class:is-open={open}>
						<span class="spectrum-TreeView-itemLink">
							<svg class="spectrum-Icon spectrum-UIIcon-ChevronRight100 spectrum-TreeView-itemIndicator" focusable="false" aria-hidden="true">
								<use xlink:href="#spectrum-css-icon-Chevron100" />
							</svg>
							<span class="spectrum-TreeView-itemLabel">{ Object.keys(_value).length + " Related Records"}</span>
						</span>
						<ul class="spectrum-TreeView spectrum-TreeView--sizeS">
							{#each _value as _row }
							<li class="spectrum-TreeView-item">
								<span class="spectrum-TreeView-itemLink">
									<span class="spectrum-TreeView-itemLabel"> {_row["primaryDisplay"]} </span>
								</span>            
							</li>
							{/each}
						</ul>
					</li>
			</ul>
		
		{:else if viewType == "list"}

			<nav>
				<ul class="spectrum-SideNav">
					{#each _value as _row }
					<li class="spectrum-SideNav-item">
						<span class="spectrum-SideNav-itemLink"> {_row["primaryDisplay"]} </span>
					</li>
					{/each}
				</ul>
			</nav>
		
		{/if}

	{:else}
		<p> No Related Records found </p>
	{/if}

	<style>

		.spectrum-TreeView {
			margin-top: 0px;
			margin-bottom: 0px;
		}
	.spectrum-TreeView-itemLink {
			height: 18px; 
			padding-top: 0px;
			padding-bottom: 0px;
			padding-left: 20px;
		}

	.spectrum-TreeView-itemLink::before {
			height: 18px;
			padding-top: 0px;
			padding-bottom: 0px;
		}

		.spectrum-TreeView-itemLabel {
			line-height: 12px;
			margin: 0px;
		}
	</style>