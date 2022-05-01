<script>
	import MapTrophies from "./components/MapTrophies.svelte";
	import { api, getCookie, setCookie } from "./extra";

	import Header from "./components/Header.svelte";

	import { onMount } from "svelte";

	import GameMap from "./components/GameMap.svelte";
	import Sidebar from "./components/Sidebar.svelte";
	import MapFishes from "./components/MapFishes.svelte";

	var gameMap,
		sidebarActive = false,
		currentMapName = "",
		mapList,
		currentMap,
		currentMapTrophies,
		currentMapFishes;

	mapList = currentMapTrophies = currentMapFishes = [];

	//INITIAL LOAD
	onMount(async () => {
		let get = await fetch(`${api}maps/`);
		let data = await get.json();

		mapList = data.results;

		init();

		//TODO REMOVE
		// updateCurrentMap({ detail: `belayariver` });
	});

	//SYNC INITIAL LOAD
	const init = () => {};

	const updateCurrentMap = async (ev) => {
		let find = mapList?.find((m) => m.name === ev.detail);
		currentMap = find;
		currentMapName = currentMap.formatted_name;
		gameMap.updateMap(currentMap);
		let get = await fetch(`${api}fishes/${currentMap.name}/trophies`);
		if (get.status == 200) {
			get = await get.json();
			currentMapTrophies = get.results;
		}

		get = await fetch(`${api}fishes/${currentMap.name}`);
		if (get.status == 200) {
			get = await get.json();

			currentMapFishes = get.results;
		}
	};

	const sidebarToggleHandler = () => {
		sidebarActive = !sidebarActive;
		document.documentElement.style.setProperty(`--sidebar-width`, `${sidebarActive ? `${200}px` : "0px"}`);
	};

	setTimeout(sidebarToggleHandler, 200);

	var trophiesToggler = false;
	var fishesToggler = false;
</script>

<Header on:sidebarToggle={sidebarToggleHandler} />

<main>
	{#if currentMapFishes.length != 0 || currentMapTrophies.length != 0}
		<div class="left">
			{#if currentMapTrophies.length != 0}
				<div class="left-item">
					<div class="left-sub-item {trophiesToggler ? `` : `hidden`}">
						<MapTrophies map_trophies={currentMapTrophies} map_trophies_filtered={currentMapTrophies} />
					</div>

					<div class="toggler" on:click={() => (trophiesToggler = !trophiesToggler)}>
						T<span style="color:{trophiesToggler ? `red` : `green`}">{trophiesToggler ? `<` : `>`}</span>
					</div>
				</div>
			{/if}
			{#if currentMapFishes.length != 0}
				<div class="left-item">
					<div class="left-sub-item {fishesToggler ? `` : `hidden`}">
						<MapFishes map_fishes={currentMapFishes} map_fishes_filtered={currentMapFishes} />
					</div>

					<div class="toggler" on:click={() => (fishesToggler = !fishesToggler)}>
						F<span style="color:{fishesToggler ? `red` : `green`}">{fishesToggler ? `<` : `>`}</span>
					</div>
				</div>
			{/if}
		</div>
	{/if}
	<div id="page">
		<div id="gmap">
			<GameMap bind:this={gameMap} />
		</div>
	</div>

	<Sidebar {currentMap} visible={sidebarActive} {mapList} on:change_map={updateCurrentMap} />
</main>

<style>
	main {
		height: calc(100% - var(--header-height));
	}
	#page {
		width: calc(100% - var(--sidebar-width));
		height: calc(100vh - var(--header-height));
		overflow-y: auto;

		display: flex;

		align-items: center;
		justify-content: center;
	}

	.left {
		height: calc(100vh - var(--header-height));
		min-width: 175px;
		width: 275px;
		align-self: baseline;
		position: absolute;
		z-index: 500;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}
	.left-item:not(:first-child) {
		margin-top: 0.5rem;
	}
	.left-item {
		width: 100%;
		display: flex;
		position: relative;
		height: 30vh;
	}
	.toggler {
		height: 6rem;
		width: 2.5rem;
		background-color: black;
		cursor: pointer;
		padding: 0.5rem;
		text-align: center;
		writing-mode: vertical-rl;
		text-orientation: upright;
		border-top-right-radius: 5px;
		border-bottom-right-radius: 5px;

		font-weight: bold;
		position: absolute;
		top: 0;
		left: 275px;
		transition: left 0.5s ease-in-out;
	}
	.left-item:first-child .left-sub-item {
		top: -65px;
	}
	.left-item:first-child .toggler {
		top: -65px;
	}

	.left-sub-item {
		position: absolute;

		transform: translateX(0);
		transition: transform 0.5s ease-in-out;
	}
	.left-sub-item.hidden {
		transform: translateX(-100%);
	}
	.left-sub-item.hidden + .toggler {
		left: 0px;
	}

	#gmap {
		z-index: 100;
		align-self: center;
	}
</style>
