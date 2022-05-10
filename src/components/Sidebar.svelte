<script>
	import { createEventDispatcher } from "svelte";

	const event = createEventDispatcher();

	export let visible = true;
	export let mapList = [];

	const onItemClicked = (newMap) => {
		event(`change_map`, newMap.name);

		if (currentMap?.name === newMap.name) {
			currentMap = null;
		} else {
			currentMap = newMap;
		}
		return;
	};

	export var currentMap = null;
</script>

<aside class="right-sidebar {visible ? `` : `hidden`}">
	<div class="maps-list-container">
		{#each mapList as map}
			<div
				id="map-{map.name}"
				passive:true
				class="maps-list-item {currentMap == map ? `active` : ``}"
				on:click={() => {
					onItemClicked(map);
				}}
			>
				{map.formatted_name}
			</div>
		{/each}
	</div>
</aside>

<style>
	.right-sidebar {
		position: fixed;
		user-select: none;
		color: white;
		top: var(--header-height);

		right: 0;
		height: 100%;

		width: 200px;
		background-color: var(--sidebar-bg);

		overflow-y: auto;
		overflow-x: hidden;
		transform: translateX(0px);
		transition: transform 0.5s ease-in-out;
	}

	.right-sidebar.hidden {
		transform: translateX(150%);
	}

	.maps-list-container {
		overflow-x: hidden;
		overflow-y: auto;
		display: table;
		border-collapse: collapse;

		height: calc(100% - var(--header-height));
		width: 100%;

		transition: height 0.5s ease-in-out;
	}

	.maps-list-item {
		background-color: transparent;
		display: table-row;
		text-align: center;
		font-size: large;
		border-top: solid 1px var(--accent-color);
	}
	.maps-list-item:nth-child(even) {
		background-color: var(--accent-dark-color);
	}

	.maps-list-item:last-child {
		border-bottom: solid 1px var(--accent-color);
	}

	.maps-list-item:hover {
		background-color: var(--contrast-dark-color);
	}

	.category-title:hover,
	.maps-list-item:hover {
		cursor: pointer;
	}

	.maps-list-item.active {
		background-color: var(--contrast-color);
		color: var(--accent-dark-color);
		font-weight: bold;
	}

	/* MOBILE DEVICES */
	@media screen and (max-width: 700px) {
		:root {
			--sidebar-width: 0px !important;
		}
		.right-sidebar {
			display: none;
		}
	}
</style>
