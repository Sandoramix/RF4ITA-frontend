<script>
	import { onMount } from "svelte";

	import GameMap from "../components/GameMap.svelte";
	import Header from "../components/Header.svelte";
	import IgFishes from "../components/IGFishes.svelte";
	import Loading from "../components/Loading.svelte";

	import Sidebar from "../components/Sidebar.svelte";
	import { api } from "../extra";

	const lang_texts = {
		ITA: {
			value: `ITA`,
			flag_link: `./images/italian_flag.png`,
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
			value: `ENG`,
			flag_link: `./images/us_flag.png`,
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
	var currentLang = lang_texts.ENG;

	export const params = {};
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
		cacheMapFishes = new Map(),
		cacheAllFishes = [];

	//STATES
	var mapListIsLoading = true,
		isChangingMap = false,
		fishesToggler = true,
		isMobile = false;

	var innerPageDiv;

	var pageHeight = 700;
	var pageWidth = 800;

	const mapSizePercentage = 0.9;
	const mapMinSize = 350;
	const mapMaxSize = 700;
	var mapSize = 400;

	onMount(async () => {
		let lang = localStorage.getItem("langUsed");
		updateLanguageTexts({ detail: lang });

		let request = await fetch(`${api}fishes/lastchange`);
		request = await request.json();
		let last_change = request.date;

		const fetch_fishes = () => {
			fetch(`${api}fishes/`)
				.then((res) => res.json())
				.then((data) => {
					cacheAllFishes = data;

					localStorage.setItem(
						`all_fishes`,
						JSON.stringify({
							date: last_change,
							fishes: data,
						}),
					);
				});
		};

		let temp = JSON.parse(localStorage.getItem(`all_fishes`));

		if (!temp) {
			fetch_fishes();
		} else {
			if (new Date() > new Date(last_change)) {
				fetch_fishes();
			} else {
				cacheAllFishes = temp.fishes;
			}
		}

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

		fetch(`${api}maps/`)
			.then((res) => res.json())
			.then((data) => {
				mapList = data;
				sidebarToggleHandler();
				mapListIsLoading = false;
				init();
			});
	});

	//-------------------------------------

	function init() {
		let map = localStorage.getItem("lastOpenedMap");
		if (map != `null`) {
			updateCurrentMap({ detail: map });
		}
	}

	function updateLanguageTexts(ev) {
		let lang = ev.detail;

		let append_text = lang == `ITA` ? `_it` : ``;

		mapList = mapList.map((m) => {
			m.default_name = m[`formatted_name${append_text}`];
			return m;
		});
		mapFishes = mapFishes.map((f) => {
			f.default_name = f[`fish_name${append_text}`];
			return f;
		});
		mapTrophies = mapTrophies.map((f) => {
			f.default_name = f[`fish_name${append_text}`];
			return f;
		});

		currentLang = lang_texts[lang] ? lang_texts[lang] : lang_texts.ENG;
		localStorage.setItem("langUsed", lang);
	}

	function updateMapFromHeader(ev) {
		updateCurrentMap(ev, true);
	}
	async function updateCurrentMap(ev, flag = false) {
		let name = ev.detail;
		mapTrophiesComponent?.clearInput();
		mapFishesComponent?.clearInput();

		if (!flag) {
			setTimeout(() => {
				HeaderComponent.manualSelect(map);
			}, 100);
		}
		let newMap = mapList.find((map) => map.name == name);

		if (name == "null" || (currentMap && newMap && currentMap.name == newMap.name)) {
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

		mapTrophies = await updateCachedData(cacheMapTrophies, name, `${api}fishes/${name}/trophies`);

		mapSpots = await updateCachedData(cacheMapSpots, name, `${api}spots/${name}`);

		mapFishes = await updateCachedData(cacheMapFishes, name, `${api}fishes/${name}`);

		GameMapComponent.updateMap(currentMap, mapSpots);
		isChangingMap = false;
	}

	async function updateCachedData(cacheSet, searchedName, fallback_api) {
		let tmp = cacheSet.get(searchedName);

		if (tmp == null) {
			let get = await fetch(fallback_api);

			get = await get.json();
			tmp = get || [];

			cacheSet.set(searchedName, tmp);
		}
		return tmp;
	}

	function sidebarToggleHandler() {
		sidebarActive = !sidebarActive;
		document.documentElement.style.setProperty(`--sidebar-width`, `${sidebarActive ? `${200}px` : "0px"}`);
	}

	function resize(ev) {
		if (!innerPageDiv) return;
		pageHeight = parseFloat(innerPageDiv.offsetHeight);
		pageWidth = parseFloat(innerPageDiv.clientWidth);
		if (pageWidth <= 700) {
			setTimeout(() => {
				HeaderComponent.manualSelect(currentMap ? currentMap.name : `null`);
			}, 100);
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
			if (focusToggled.trophies || focusToggled.fishes) return;
			switch (ev.key.toLocaleLowerCase()) {
				case "m":
					sidebarToggleHandler();
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

	var focusToggled = {
		trophies: false,
		fishes: false,
	};
	function focusToggler(flag, variable) {
		focusToggled[variable] = flag;
	}
</script>

<svelte:window on:resize={resize} />

<Header
	currentLanguage={currentLang}
	on:change_language={updateLanguageTexts}
	bind:this={HeaderComponent}
	on:sidebarToggle={sidebarToggleHandler}
	{mapList}
	on:change_map={updateMapFromHeader}
/>

{#if mapListIsLoading}
	<Loading />
{/if}
<div class="left">
	{#if cacheAllFishes.length != 0 || mapTrophies.length != 0 || mapFishes.length != 0}
		<div class="left-item">
			<div class="left-sub-item {fishesToggler ? `` : `hidden`}">
				<!-- <SidebarMapFishes
						on:focus_toggle={(ev) => focusToggler(ev, `fishes`)}
						placeholder={currentLang.search_fish}
						title={currentLang.map_fishes}
						bind:this={mapFishesComponent}
						fishes={mapFishes}
						fishes_filtered={mapFishes}
					/> -->
				<IgFishes allFishes={cacheAllFishes} {mapFishes} {mapTrophies} />
			</div>

			<div class="toggler" passive:true on:click={() => (fishesToggler = !fishesToggler)}>
				F<span style="color:whitesmoke;">{fishesToggler ? `🡰` : `🡲`}</span>
			</div>
		</div>
	{/if}
</div>
<div id="page" bind:this={innerPageDiv}>
	{#if isChangingMap}
		<Loading />
	{/if}
	<div id="gmap">
		<GameMap {currentLang} on:can_resize={onMapLoaded} bind:this={GameMapComponent} {isMobile} />
	</div>
</div>

<Sidebar {currentMap} visible={sidebarActive} {mapList} on:change_map={updateCurrentMap} />

<style>
	.homepage-footer {
		position: absolute;
		bottom: 0;
		left: 0;
		display: flex;
		width: calc(100vw - var(--sidebar-width) - 0.5rem);
		height: 100px;
		display: flex;
		justify-content: center;
	}

	#page {
		width: calc(100% - var(--sidebar-width));

		height: calc(100vh - var(--header-height));
		overflow: auto;

		position: relative;
		display: flex;

		transition: width 0.4s ease-in-out;
		align-items: center;
		justify-content: center;
	}

	.left {
		height: calc(100vh - var(--header-height));

		position: fixed;
		left: 0;
		top: var(--header-height);
		z-index: 500;
		display: flex;
		flex-direction: column;
		justify-content: center;
		/* gap: 0.1rem; */
	}
	/* .left-item:not(:first-child) {
		/* margin-top: 0.5rem; 
	} */
	.left-item {
		display: flex;
		position: relative;
		height: 50%;
	}
	.left-item > * {
		width: 275px;
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
		align-self: center;
	}
</style>
