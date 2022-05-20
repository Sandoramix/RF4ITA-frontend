<script>
	import GameMap from "../components/GameMap.svelte";
	import Header from "../components/Header.svelte";
	import Loading from "../components/Loading.svelte";

	import Sidebar from "../components/Sidebar.svelte";
	import SidebarMapFishes from "../components/SidebarMapFishes.svelte";
	import { api } from "../extra";

	const langTexts = {
		ITA: {
			map_trophies: `Trofei della mappa`,
			map_fishes: `Pesci della mappa`,
			maps: `Mappe`,
			coords: `Coordinate`,
			search_coords: `Cerca coordinate`,
			distance: `Distanza`,
			active_spots: `Spot attivi`,
			ground: `Tipo di fondale`,
			search_fish: `Cerca il pesce`,
			choose_map: `Scegli la mappa`,
			ig_clock: `Ora IG`,
		},
		ENG: {
			map_trophies: `Map Trophies`,
			map_fishes: `All map fishes`,
			maps: `Maps`,
			coords: `Coords`,
			search_coords: `Search coords`,
			distance: `Distance`,
			active_spots: `Active spots`,
			ground: `Ground`,
			search_fish: `Search for fish`,
			choose_map: `Choose the map`,
			ig_clock: `IG Clock`,
		},
	};
	var currentLang = langTexts.ITA;

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
	const mapMinSize = 350;
	const mapMaxSize = 700;
	var mapSize = 400;

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
		let lang = localStorage.getItem("langUsed");
		HeaderComponent.manualSetLanguage(lang);
		if (map != `null`) {
			updateCurrentMap({ detail: map });
		}
	}

	function handleLanguageUpdate(ev) {
		let lang = ev.detail;

		const engCase = () => {
			mapList = mapList.map((m) => {
				m.default_name = m.formatted_name;
				return m;
			});
			mapFishes = mapFishes.map((f) => {
				f.default_name = f.fish_name;
				return f;
			});
			mapTrophies = mapTrophies.map((f) => {
				f.default_name = f.fish_name;
				return f;
			});

			currentLang = langTexts.ENG;
		};

		switch (lang) {
			case "ITA":
				mapList = mapList.map((m) => {
					m.default_name = m.formatted_name_it;
					return m;
				});
				mapFishes = mapFishes.map((f) => {
					f.default_name = f.fish_name_it;
					return f;
				});
				mapTrophies = mapTrophies.map((f) => {
					f.default_name = f.fish_name_it;
					return f;
				});
				currentLang = langTexts.ITA;
				break;
			case "ENG":
				engCase();
				break;
			default:
				engCase();
				break;
		}
		localStorage.setItem("langUsed", lang);
	}

	function mobileUpdateSelectedMap(map, timeout) {
		setTimeout(() => {
			HeaderComponent.manualSelect(map);
		}, timeout);
	}
	function updateMapFromHeader(ev) {
		updateCurrentMap(ev, true);
	}
	async function updateCurrentMap(ev, flag) {
		let name = ev.detail;
		mapTrophiesComponent?.clearInput();
		mapFishesComponent?.clearInput();

		if (!flag) {
			mobileUpdateSelectedMap(name, 100);
		}
		let newMap = mapList.find((map) => map.name == name);

		if (name === "null" || (currentMap && newMap && currentMap.name == newMap.name)) {
			currentMap = null;
			mapTrophies = [];
			mapFishes = [];
			mapSpots = [];
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
		handleLanguageUpdate({ detail: localStorage.getItem("langUsed") });
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
		if (pageWidth <= 700) {
			mobileUpdateSelectedMap(currentMap ? currentMap.name : `null`, 100);
		}
		mapSize = Math.round(pageHeight < pageWidth ? pageHeight * (mapSizePercentage - 0.07) : pageWidth * mapSizePercentage);
		mapSize = mapSize < mapMinSize ? mapMinSize : mapSize;
		mapSize = mapSize > mapMaxSize ? mapMaxSize : mapSize;

		GameMapComponent.updateSize(mapSize);
	}
	function onMapLoaded() {
		resize(null);
		HeaderComponent.manualSelect(map);
	}
	document.addEventListener(`resize`, resize, { passive: true });
	document.addEventListener(
		`keydown`,
		(ev) => {
			switch (ev.key.toLocaleLowerCase()) {
				case "m":
					sidebarToggleHandler();
					break;
				case "t":
					trophiesToggler = !trophiesToggler;
					break;
				case "f":
					fishesToggler = !fishesToggler;
					break;

				case "g":
					GameMapComponent.groundToggle();
					break;
				case "a":
					GameMapComponent.activeSpotsToggle();
					break;
			}
		},
		{ passive: true },
	);
</script>

<svelte:window on:resize={resize} />

<Header
	currentLangTexts={currentLang}
	on:change_language={handleLanguageUpdate}
	bind:this={HeaderComponent}
	on:sidebarToggle={sidebarToggleHandler}
	{mapList}
	on:change_map={updateMapFromHeader}
/>

{#if mapListIsLoading}
	<Loading />
{/if}
<div class="left">
	{#if mapTrophies.length != 0 || mapFishes.length != 0}
		{#if mapTrophies.length != 0}
			<div class="left-item">
				<div class="left-sub-item {trophiesToggler ? `` : `hidden`}">
					<SidebarMapFishes
						placeholder={currentLang.search_fish}
						title={currentLang.map_trophies}
						bind:this={mapTrophiesComponent}
						fishes={mapTrophies}
						fishes_filtered={mapTrophies}
					/>
				</div>

				<div class="toggler" passive:true on:click={() => (trophiesToggler = !trophiesToggler)}>
					T<span style="color:whitesmoke;">{trophiesToggler ? `🡰` : `🡲`}</span>
				</div>
			</div>
		{/if}
		{#if mapFishes.length != 0}
			<div class="left-item">
				<div class="left-sub-item {fishesToggler ? `` : `hidden`}">
					<SidebarMapFishes
						placeholder={currentLang.search_fish}
						title={currentLang.map_fishes}
						bind:this={mapFishesComponent}
						fishes={mapFishes}
						fishes_filtered={mapFishes}
					/>
				</div>

				<div class="toggler" passive:true on:click={() => (fishesToggler = !fishesToggler)}>
					F<span style="color:whitesmoke;">{fishesToggler ? `🡰` : `🡲`}</span>
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
		<GameMap {currentLang} on:can_resize={onMapLoaded} bind:this={GameMapComponent} {isMobile} />
	</div>
	<footer class="homepage-footer">
		<ul><li><a href="#/policy" rel="noopener noreferrer">Policy</a></li></ul>
	</footer>
</div>

<Sidebar {currentMap} visible={sidebarActive} {mapList} on:change_map={updateCurrentMap} />

<style>
	.homepage-footer {
		position: absolute;
		bottom: 0.5rem;
		left: 0.5rem;
	}
	.homepage-footer ul {
		list-style: none;
	}
	.homepage-footer a {
		color: var(--primary-color);
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
		background-color: var(--red-dark-color);
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
		background-color: var(--red-color);
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
