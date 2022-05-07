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
		mapTrophies,
		mapFishes,
		mapSpots;

	mapList = mapSpots = mapTrophies = mapFishes = [];

	//INITIAL LOAD
	onMount(async () => {
		fetch(`${api}ismobile`)
			.then((res) => res.json())
			.then((res) => {
				if (res.result) {
					alert(
						`This site is meant to be used on computer browsers.\nTo have an better experience we recommend you to not use it from mobile phones`,
					);
				}
			});
		let get = await fetch(`${api}maps/`);
		let data = await get.json();

		mapList = data.results;

		init();

		//TODO REMOVE
		updateCurrentMap({ detail: `thecottagepond` });
	});

	//SYNC INITIAL LOAD
	const init = () => {};

	const updateCurrentMap = async (ev) => {
		let find = mapList?.find((m) => m.name === ev.detail);
		if (!find) return;

		currentMap = find;
		currentMapName = currentMap.formatted_name;

		let get = await fetch(`${api}fishes/${currentMap.name}/trophies`);
		if (get.status == 200) {
			get = await get.json();
			mapTrophies = get.results || [];
		}

		get = await fetch(`${api}spots/${currentMap.name}`);
		if (get.status == 200) {
			get = await get.json();

			mapSpots = get.results || [];
			console.log(mapSpots);
		}

		get = await fetch(`${api}fishes/${currentMap.name}`);
		if (get.status == 200) {
			get = await get.json();

			mapFishes = get.results || [];
		}

		gameMap.updateMap(currentMap);
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
	{#if mapFishes.length != 0 || mapTrophies.length != 0}
		<div class="left">
			{#if mapTrophies.length != 0}
				<div class="left-item">
					<div class="left-sub-item {trophiesToggler ? `` : `hidden`}">
						<MapTrophies map_trophies={mapTrophies} map_trophies_filtered={mapTrophies} />
					</div>

					<div class="toggler" on:click={() => (trophiesToggler = !trophiesToggler)}>
						T<span style="color:{trophiesToggler ? `red` : `green`}">{trophiesToggler ? `🡰` : `🡲`}</span>
					</div>
				</div>
			{/if}
			{#if mapFishes.length != 0}
				<div class="left-item">
					<div class="left-sub-item {fishesToggler ? `` : `hidden`}">
						<MapFishes map_fishes={mapFishes} map_fishes_filtered={mapFishes} />
					</div>

					<div class="toggler" on:click={() => (fishesToggler = !fishesToggler)}>
						F<span style="color:{fishesToggler ? `red` : `green`}">{fishesToggler ? `🡰` : `🡲`}</span>
					</div>
				</div>
			{/if}
		</div>
	{/if}
	<div id="page">
		<div id="gmap">
			<GameMap bind:this={gameMap} {mapSpots} />
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

		align-self: baseline;
		position: absolute;
		z-index: 500;
		display: flex;
		flex-direction: column;
		justify-content: center;
		gap: 0.5rem;
	}
	.left-item:not(:first-child) {
		margin-top: 0.5rem;
	}
	.left-item {
		display: flex;
		position: relative;
		height: 30vh;
	}
	.toggler {
		height: 6rem;
		width: 2.5rem;
		background-color: var(--header-bg);
		border-right: 1px solid black;

		border-top: 1px solid black;

		border-bottom: 1px solid black;
		cursor: pointer;
		padding: 0.5rem;
		text-align: center;
		writing-mode: vertical-rl;
		text-orientation: upright;
		border-top-right-radius: 5px;
		border-bottom-right-radius: 5px;
		user-select: none;
		font-weight: bold;
		position: absolute;
		top: 0;
		left: var(--map-left-panel-width);
		transition: left 0.5s ease-in-out;
	}
	.toggler:hover {
		background-color: #5300a0;
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
