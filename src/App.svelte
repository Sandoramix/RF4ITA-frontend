<script>
	import Loading from "./components/Loading.svelte";
	import MapTrophies from "./components/MapTrophies.svelte";
	import { api } from "./extra";

	import Header from "./components/Header.svelte";

	import GameMap from "./components/GameMap.svelte";
	import Sidebar from "./components/Sidebar.svelte";
	import MapFishes from "./components/MapFishes.svelte";

	var currentMap,
		gameMap,
		sidebarActive = false;

	var mapTrophies = [],
		mapFishes = [],
		mapSpots = [];

	var mapList = [];

	var cacheMapSpots = new Map(),
		cacheMapTrophies = new Map(),
		cacheMapFishes = new Map();

	var mapListIsLoading = true,
		isChangingMap = false;
	//INITIAL LOAD
	fetch(`${api}ismobile`)
		.then((res) => res.json())
		.then((res) => {
			if (res.result) {
				alert(
					`This site is meant to be used on computer browsers.\nTo have an better experience we recommend you to not use it from mobile phones`,
				);
			}
		});
	function fetchMaps() {
		fetch(`${api}maps/`)
			.then((res) => res.json())
			.then((data) => {
				mapList = data.results;
				mapListIsLoading = false;
				init();
			})
			.catch((err) => {
				fetchMaps();
			});
	}
	fetchMaps();
	//-------------------------------------

	//SYNC INITIAL LOAD
	const init = () => {
		let map = localStorage.getItem("lastOpenedMap");

		if (map != `null`) {
			updateCurrentMap({ detail: map });
		}
	};

	const updateCurrentMap = async (ev) => {
		let name = ev.detail;
		mapTrophiesObj?.clearInput();
		mapFishesObj?.clearInput();
		let newMap = mapList.find((map) => map.name == name);

		if (currentMap && newMap && currentMap.name == newMap.name) {
			currentMap = null;
			mapTrophies = [];
			mapFishes = [];
			mapSpots = [];

			localStorage.setItem("lastOpenedMap", null);
			gameMap.removeMap();
			return;
		}
		currentMap = newMap;

		if (!currentMap) return;
		isChangingMap = true;
		localStorage.setItem("lastOpenedMap", name);
		var get;

		let mapTrophies_tmp = cacheMapTrophies.get(name);

		if (mapTrophies_tmp == null) {
			get = await fetch(`${api}fishes/${name}/trophies`);

			get = await get.json();
			mapTrophies_tmp = get.results || [];

			cacheMapTrophies.set(name, mapTrophies_tmp);
		}
		mapTrophies = mapTrophies_tmp;

		let mapSpots_tmp = cacheMapSpots.get(name);

		if (mapSpots_tmp == null) {
			get = await fetch(`${api}spots/${name}`);

			get = await get.json();
			mapSpots_tmp = get.results || [];

			cacheMapSpots.set(name, mapSpots_tmp);
		}
		mapSpots = mapSpots_tmp;

		let mapFishes_tmp = cacheMapFishes.get(name);
		if (mapFishes_tmp == null) {
			get = await fetch(`${api}fishes/${name}`);

			get = await get.json();
			mapFishes_tmp = get.results || [];

			cacheMapFishes.set(name, mapFishes_tmp);
		}
		mapFishes = mapFishes_tmp;

		gameMap.updateMap(currentMap, mapSpots);
		isChangingMap = false;
	};

	const sidebarToggleHandler = () => {
		sidebarActive = !sidebarActive;
		document.documentElement.style.setProperty(`--sidebar-width`, `${sidebarActive ? `${200}px` : "0px"}`);
	};

	var mapTrophiesObj, mapFishesObj;

	setTimeout(sidebarToggleHandler, 200);
	var trophiesToggler = false;
	var fishesToggler = false;
</script>

<Header on:sidebarToggle={sidebarToggleHandler} />

<main>
	{#if mapListIsLoading}
		<Loading />
	{/if}

	<div class="left">
		{#if mapTrophies.length != 0 || mapFishes.length != 0}
			{#if mapTrophies.length != 0}
				<div class="left-item">
					<div class="left-sub-item {trophiesToggler ? `` : `hidden`}">
						<MapTrophies bind:this={mapTrophiesObj} map_trophies={mapTrophies} map_trophies_filtered={mapTrophies} />
					</div>

					<div class="toggler" passive:true on:click={() => (trophiesToggler = !trophiesToggler)}>
						T<span style="color:{trophiesToggler ? `red` : `green`}">{trophiesToggler ? `🡰` : `🡲`}</span>
					</div>
				</div>
			{/if}
			{#if mapFishes.length != 0}
				<div class="left-item">
					<div class="left-sub-item {fishesToggler ? `` : `hidden`}">
						<MapFishes bind:this={mapFishesObj} map_fishes={mapFishes} map_fishes_filtered={mapFishes} />
					</div>

					<div class="toggler" passive:true on:click={() => (fishesToggler = !fishesToggler)}>
						F<span style="color:{fishesToggler ? `red` : `green`}">{fishesToggler ? `🡰` : `🡲`}</span>
					</div>
				</div>
			{/if}
		{/if}
	</div>

	<div id="page">
		{#if isChangingMap}
			<Loading />
		{/if}
		<div id="gmap">
			<GameMap bind:this={gameMap} />
		</div>
	</div>

	<Sidebar {currentMap} visible={sidebarActive} {mapList} on:change_map={updateCurrentMap} />
</main>

<style>
	main {
		height: calc(100% - var(--header-height));
		position: relative;
	}
	#page {
		width: calc(100% - var(--sidebar-width));
		height: calc(100vh - var(--header-height));
		overflow-y: auto;
		position: relative;
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
