<script>
	import Loading from "./components/Loading.svelte";
	import MapTrophies from "./components/MapTrophies.svelte";
	import { api } from "./extra";

	import Header from "./components/Header.svelte";

	import GameMap from "./components/GameMap.svelte";
	import Sidebar from "./components/Sidebar.svelte";
	import MapFishes from "./components/MapFishes.svelte";

	//COMPONENTS
	var GameMapComponent, HeaderComponent, mapTrophiesComponent, mapFishesComponent;

	//MAP OBJECTS
	var currentMap,
		//SIDEBAR STATE
		sidebarActive = false;

	//CURRENT MAP'S FETCHED DATA
	var mapTrophies = [],
		mapFishes = [],
		mapSpots = [];

	//LIST OF ALL MAPS
	var mapList = [];

	//CACHE DATA GATHERED FROM BACKEND
	var cacheMapSpots = new Map(),
		cacheMapTrophies = new Map(),
		cacheMapFishes = new Map();

	//STATES
	var mapListIsLoading = true,
		isChangingMap = false,
		trophiesToggler = false,
		fishesToggler = false,
		isMobile = false;

	var innerPageDiv;

	var pageHeight = 800;
	var pageWidth = 1000;

	const mapSizePercentage = 0.9;
	const mapMinSize = 320;

	var mapSize = 400;

	//INITIAL LOAD
	fetch(`${api}ismobile`)
		.then((res) => res.json())
		.then((res) => {
			if (res.result) {
				isMobile = true;
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
	function init() {
		let map = localStorage.getItem("lastOpenedMap");

		if (map != `null`) {
			updateCurrentMap({ detail: map });
		}
		// mobileUpdateSelectedMap(map,100)
	}

	function mobileUpdateSelectedMap(map, timeout) {
		setTimeout(() => {
			HeaderComponent.manualSelect(map);
		}, timeout);
	}

	async function updateCurrentMap(ev) {
		let name = ev.detail;
		mapTrophiesComponent?.clearInput();
		mapFishesComponent?.clearInput();
		mobileUpdateSelectedMap(name, 100);
		let newMap = mapList.find((map) => map.name == name);

		if (name === "null" || (currentMap && newMap && currentMap.name == newMap.name)) {
			currentMap = null;
			mapTrophies = [];
			mapFishes = [];
			mapSpots = [];
			mobileUpdateSelectedMap(name, 100);
			localStorage.setItem("lastOpenedMap", null);
			GameMapComponent.removeMap();

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

		GameMapComponent.updateMap(currentMap, mapSpots);
		isChangingMap = false;
	}

	function sidebarToggleHandler() {
		sidebarActive = !sidebarActive;
		document.documentElement.style.setProperty(`--sidebar-width`, `${sidebarActive ? `${200}px` : "0px"}`);
	}

	sidebarToggleHandler();

	function resize(ev) {
		if (!innerPageDiv) return;
		pageHeight = parseFloat(innerPageDiv.offsetHeight);
		pageWidth = parseFloat(innerPageDiv.clientWidth);

		mapSize = Math.round(pageHeight < pageWidth ? pageHeight * mapSizePercentage : pageWidth * mapSizePercentage);
		mapSize = mapSize < mapMinSize ? mapMinSize : mapSize;
		GameMapComponent.updateSize(mapSize);
	}
	function onMapLoaded() {
		resize(null);
		HeaderComponent.manualSelect(map);
	}
	document.addEventListener(`resize`, resize, { passive: true });
</script>

<svelte:window on:resize={resize} />

<Header bind:this={HeaderComponent} on:sidebarToggle={sidebarToggleHandler} {mapList} on:change_map={updateCurrentMap} />

<main>
	{#if mapListIsLoading}
		<Loading />
	{/if}
	<div class="left">
		{#if mapTrophies.length != 0 || mapFishes.length != 0}
			{#if mapTrophies.length != 0}
				<div class="left-item">
					<div class="left-sub-item {trophiesToggler ? `` : `hidden`}">
						<MapTrophies bind:this={mapTrophiesComponent} map_trophies={mapTrophies} map_trophies_filtered={mapTrophies} />
					</div>

					<div class="toggler" passive:true on:click={() => (trophiesToggler = !trophiesToggler)}>
						T<span style="color:{trophiesToggler ? `red` : `green`}">{trophiesToggler ? `🡰` : `🡲`}</span>
					</div>
				</div>
			{/if}
			{#if mapFishes.length != 0}
				<div class="left-item">
					<div class="left-sub-item {fishesToggler ? `` : `hidden`}">
						<MapFishes bind:this={mapFishesComponent} map_fishes={mapFishes} map_fishes_filtered={mapFishes} />
					</div>

					<div class="toggler" passive:true on:click={() => (fishesToggler = !fishesToggler)}>
						F<span style="color:{fishesToggler ? `red` : `green`}">{fishesToggler ? `🡰` : `🡲`}</span>
					</div>
				</div>
			{/if}
		{/if}
	</div>
	<div id="page" bind:this={innerPageDiv}>
		{#if isChangingMap}
			<Loading />
		{/if}
		<div id="gmap">
			<GameMap on:can_resize={onMapLoaded} bind:this={GameMapComponent} {isMobile} />
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
		background-color: #850000;
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
		background-color: #db0000;
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
		margin-top: 0.5rem;
		z-index: 100;
		align-self: flex-start;
	}
</style>
