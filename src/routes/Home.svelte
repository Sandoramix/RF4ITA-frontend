<script>
	import { Base64 } from "js-base64";

	import { onMount } from "svelte";
	export let params = {};
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
			fish: `Pesci`,
			this_map_trophies: `Trofei della mappa`,
			this_map_fish: `Pesci della mappa`,
			search_fish: `Cerca il pesce`,

			maps: `Mappe`,
			coords: `Coordinate`,
			search_coords: `Cerca coordinate`,
			distance: `Distanza`,
			active_spots: `Spot attivi`,
			ground: `Tipo di fondale`,

			choose_map: `Scegli la mappa`,
			ig_clock: `Ora IG`,
		},
		ENG: {
			value: `ENG`,
			flag_link: `./images/us_flag.png`,
			fish: `Fish`,
			this_map_trophies: `Map trophies`,
			this_map_fish: `Map fish`,
			search_fish: `Search fish`,
			maps: `Maps`,
			coords: `Coords`,
			search_coords: `Search coords`,
			distance: `Distance`,
			active_spots: `Active spots`,
			ground: `Ground`,

			choose_map: `Choose the map`,
			ig_clock: `IG Clock`,
		},
	};
	var currentLang = lang_texts.ENG;

	//COMPONENTS
	var GameMapComponent, HeaderComponent, IGFishesComponent;

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
		fishesToggler = false,
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
					cacheAllFishes = [...data];

					localStorage.setItem(
						`all_fishes`,
						JSON.stringify({
							date: last_change,
							fishes: Base64.encode(JSON.stringify(data)),
						}),
					);
				});
		};
		let x = JSON.parse(localStorage.getItem(`all_fishes`));

		if (!x) {
			fetch_fishes();
		} else {
			if (new Date(x.date) < new Date(last_change)) {
				fetch_fishes();
			} else {
				x.fishes = JSON.parse(Base64.decode(x.fishes));
				cacheAllFishes = x.fishes;
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

		currentLang = lang_texts[lang] ? lang_texts[lang] : lang_texts.ENG;
		localStorage.setItem("langUsed", lang);
	}

	function updateMapFromHeader(ev) {
		updateCurrentMap(ev, true);
	}
	async function updateCurrentMap(ev, flag = false) {
		let name = ev.detail;
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
					IGFishesComponent.toggle();
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

<IgFishes bind:this={IGFishesComponent} allFishes={cacheAllFishes} {mapFishes} {mapTrophies} currentLanguage={currentLang} />

<div id="page" bind:this={innerPageDiv}>
	{#if isChangingMap}
		<Loading />
	{/if}
	<div id="gmap">
		<GameMap {currentLang} on:can_resize={onMapLoaded} bind:this={GameMapComponent} {isMobile} />
	</div>
</div>

<Sidebar currentLanguage={currentLang} {currentMap} visible={sidebarActive} {mapList} on:change_map={updateCurrentMap} />

<style>
	/* .homepage-footer {
		position: absolute;
		bottom: 0;
		left: 0;
		display: flex;
		width: calc(100vw - var(--sidebar-width) - 0.5rem);
		height: 100px;
		display: flex;
		justify-content: center;
	} */

	#page {
		width: calc(100% - var(--sidebar-width));

		height: calc(100vh - var(--header-height));
		overflow: auto;

		position: relative;
		display: flex;

		transition: width 0.3s ease-in-out;
		align-items: center;
		justify-content: center;
	}

	#gmap {
		margin-top: 0.5rem;

		align-self: center;
	}
</style>
