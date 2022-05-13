<script>
	import DrawPanel from ".\\DrawPanel.svelte";
	import * as L from "leaflet";
	import { createEventDispatcher, onMount } from "svelte";
	import { api } from "..\\extra";
	const dispatcher = createEventDispatcher();
	var IMG_OUTER_SIZE = 1024,
		IMG_MIN_INNER_PX = 40,
		IMG_MAX_INNER_PX = 986,
		OUTER_BOUNDS = [
			[0, 0],
			[IMG_OUTER_SIZE, IMG_OUTER_SIZE],
		],
		// INNER_BOUNDS = [
		// 	[IMG_MIN_INNER_PX, IMG_MIN_INNER_PX],
		// 	[IMG_MAX_INNER_PX, IMG_MAX_INNER_PX],
		// ],
		IMG_INNER_SIZE = IMG_OUTER_SIZE - IMG_MIN_INNER_PX * 2,
		IG_MAP_SQUARE_LENGTH = 94;

	//ELEMENTS
	var DRAW_PANEL, SEARCH_LATLNG_BTN, SEARCH_LNG, SEARCH_LAT, MAP_CNT, LEAFLET_MAP, currCoordsSpan, currDistanceDiv;

	var currentMap,
		leafletMap,
		mapTileLayer,
		xRatio = 0,
		yRatio = 0,
		x = 0,
		y = 0,
		userFirstMark = L.circleMarker(),
		userSecondMark = L.circleMarker(),
		userMarksLine = L.polyline([
			[-1, -1],
			[-1, -1],
		]);
	const minZoom = 0,
		maxZoom = 2;
	userFirstMark = userSecondMark = userMarksLine = null;

	export var isMobile = false;

	onMount(() => {
		SEARCH_LATLNG_BTN.addEventListener(`click`, () => searchCoordsHandler(true), { passive: true });
	});
	export function removeMap() {
		if (leafletMap) {
			leafletMap.invalidateSize();
			leafletMap.off();
			leafletMap.remove();
		}
		LEAFLET_MAP.innerHTML = `<div id="map"></div>`;
		leafletMap = null;
		currentMap = null;
		userFirstMark = userSecondMark = userMarksLine = null;
	}
	export async function updateMap(map, mapSpots) {
		removeMap();
		if (!map) return;
		currentMap = map;
		xRatio = IMG_INNER_SIZE / (currentMap.limits.x.max - currentMap.limits.x.min);
		yRatio = IMG_INNER_SIZE / (currentMap.limits.y.max - currentMap.limits.y.min);

		mapTileLayer = L.tileLayer(`${api}maps/${currentMap.name}/{z}_{x}_{y}.jpg`, {
			minZoom,
			maxZoom,
			noWrap: true,
			attribution: 'Map inspired by &copy; <a href="https://rf4.info">rf4.info</a>',
		});

		var customCRS = L.extend(L.CRS.Simple, {
			projection: L.extend(L.Projection.LonLat, {
				bounds: L.bounds([0, 0], [IMG_OUTER_SIZE, IMG_OUTER_SIZE]),
			}),
			transformation: new L.Transformation(1, 0, 1, 0),
			scale: function (zoom) {
				return Math.pow(2, zoom - 2);
			},
			infinite: false,
		});

		let obj = document.createElement("object");
		obj.type = "image/svg+xml";

		var allLayers = [mapTileLayer];

		leafletMap = L.map(`map`, {
			crs: customCRS,
			layers: allLayers,
			maxBounds: OUTER_BOUNDS,
			doubleClickZoom: false,
		}).setView([0, 0], 1);

		let ctrl = L.control.layers({}, {}).setPosition("topleft");
		let canAddCtrl = false;

		let mapSpotsMarkers = [];

		mapSpots.forEach((spot) => {
			let mapX = gameToMapCoord(spot.x, xRatio, currentMap.limits.x),
				mapY = gameToMapCoord(spot.y, yRatio, currentMap.limits.y, true);

			let spotMarker = newCircleMarker(L.latLng(mapY, mapX), 4, `${spot.x}:${spot.y}`, `#ffe600`, `#e67a00`, `top`);
			mapSpotsMarkers.push(spotMarker);
		});

		if (mapSpotsMarkers.length > 0) {
			let spotsLayer = L.layerGroup(mapSpotsMarkers).addTo(leafletMap);

			ctrl.addOverlay(spotsLayer, "Active spots");
			canAddCtrl = true;
		}

		let get = await fetch(`${api}maps/${currentMap.name}/ground`);

		if (get.status != 404) {
			obj.data = `${api}maps/${currentMap.name}/ground`;

			var grOverlay = L.svgOverlay(obj, OUTER_BOUNDS).addTo(leafletMap);

			ctrl.addOverlay(grOverlay, "Ground");
			canAddCtrl = true;
		}
		if (canAddCtrl) {
			ctrl.addTo(leafletMap);
		}

		leafletMap.on("click", onLeafletMapClick);

		leafletMap.on("mousemove", (ev) => {
			y = IMG_OUTER_SIZE - ev.latlng.lat;
			x = ev.latlng.lng;

			let check = isCoordsInBounds(x, y);

			let igX = mapToGameCoord(x, xRatio, currentMap.limits.x);
			let igY = mapToGameCoord(y, yRatio, currentMap.limits.y);

			currCoordsSpan.textContent = `${check ? `${igX}:${igY}` : `-:-`}`;
			if (!check) return;
			if (userFirstMark && !userSecondMark) {
				updatePolyline(ev.latlng);
			}
		});
		dispatcher("can_resize");
		leafletMap.invalidateSize();
	}
	function onLeafletMapClick(ev) {
		y = ev.latlng.lat;
		x = ev.latlng.lng;

		if (!isCoordsInBounds(x, y)) return;

		createUserMarkerHandler(ev.latlng);
	}
	function newCircleMarker(latlng, radius = 10, tooltipContent = null, color = `#3388ff`, fill = null, direction = `auto`) {
		let marker = L.circleMarker(latlng, { radius, color, fillColor: fill });
		if (tooltipContent) {
			let tip = L.tooltip({ direction, interactive: false }).setContent(tooltipContent);
			marker.bindTooltip(tip);
		}

		return marker;
	}

	function mapToGameCoord(coord, ratio, limits) {
		return Math.round((coord - IMG_MIN_INNER_PX) / ratio + limits.min);
	}
	function gameToMapCoord(coord, ratio, limits, isY = false) {
		let tmp = (coord - limits.min) * ratio + IMG_MIN_INNER_PX;
		return isY ? IMG_OUTER_SIZE - tmp : tmp;
	}

	function isCoordsInBounds(x, y) {
		return x >= IMG_MIN_INNER_PX && x <= IMG_MAX_INNER_PX && y >= IMG_MIN_INNER_PX && y <= IMG_MAX_INNER_PX;
	}

	function searchCoordsHandler(flag = false) {
		let igX = parseInt(SEARCH_LNG.value);
		let igY = parseInt(SEARCH_LAT.value);

		let goodCoords = searchCoordInBounds(igX, currentMap.limits.x, SEARCH_LNG);
		goodCoords = goodCoords && searchCoordInBounds(igY, currentMap.limits.y, SEARCH_LAT);

		if (!goodCoords) return;

		SEARCH_LNG.value = ``;
		SEARCH_LAT.value = ``;

		let mapX = gameToMapCoord(igX, xRatio, currentMap.limits.x),
			mapY = gameToMapCoord(igY, yRatio, currentMap.limits.y, true);
		if (!isCoordsInBounds(mapX, mapY)) return;

		if (userSecondMark) {
			createUserMarkerHandler(L.latLng(mapY, mapX));
		}
		createUserMarkerHandler(L.latLng(mapY, mapX));
	}

	function createUserMarkerHandler(latlng) {
		if (userFirstMark && userSecondMark) {
			userFirstMark.removeFrom(leafletMap);
			userSecondMark.removeFrom(leafletMap);
			userMarksLine.removeFrom(leafletMap);
			currDistanceDiv.innerHTML = ``;
			userFirstMark = userSecondMark = userMarksLine = null;
			return;
		}

		let igx = mapToGameCoord(latlng.lng, xRatio, currentMap.limits.x),
			igy = mapToGameCoord(IMG_OUTER_SIZE - latlng.lat, yRatio, currentMap.limits.y);

		if (userFirstMark) {
			userSecondMark = newCircleMarker(latlng, 5, `${igx}:${igy}`, `#40FF40`).addTo(leafletMap);
			updatePolyline(latlng);

			return;
		}
		userFirstMark = newCircleMarker(latlng, 7, `${igx}:${igy}`).addTo(leafletMap);

		currDistanceDiv.textContent = `Distance: 0m`;

		userMarksLine = L.polyline([userFirstMark.getLatLng(), latlng], {}).addTo(leafletMap);
	}

	function updatePolyline(latlng, offset = 0) {
		if (!userFirstMark) return;
		// let midpoint = midpointBetweenLatlng(userFirstMark.getLatLng(), latlng);

		let direction = getOrientation(latlng);

		userMarksLine.setLatLngs([userFirstMark.getLatLng(), latlng]);

		let distance = distanceBetweenPoints(userFirstMark.getLatLng(), latlng, IG_MAP_SQUARE_LENGTH / currentMap.square_distance);
		currDistanceDiv.innerHTML = `<span id="distance-prefix">Distance: </span><span id="distance"><b>${distance}m</b> <em><b>${
			distance !== 0 ? direction : ``
		}</b></em>`;
	}

	function getOrientation(latlng2) {
		let currDirection = "";
		if (!userFirstMark) return currDirection;
		let latlng1 = userFirstMark.getLatLng();
		currDirection += latlng1.lat < latlng2.lat ? "S" : "N";
		currDirection += latlng1.lng < latlng2.lng ? "E" : "W";
		return currDirection;
	}

	function distanceBetweenPoints(point_a, point_b, divider) {
		return Math.round(Math.sqrt(Math.pow(point_b.lat - point_a.lat, 2) + Math.pow(point_b.lng - point_a.lng, 2)) / divider);
	}

	function searchCoordInBounds(coord, limits, element) {
		if (coord > limits.max || coord < limits.min) {
			element.classList.add(`red-border`);
			return false;
		}

		element.classList.remove(`red-border`);
		return true;
	}

	//DISABLE DRAGGING WHEN IN DRAW MODE
	const mapDragToggler = () => {
		if (leafletMap.dragging.enabled()) {
			return leafletMap.dragging.disable();
		}
		leafletMap.dragging.enable();
	};

	//MAP SIZING
	export function updateSize(size) {
		MAP_CNT.style.width = `${size}px`;
		MAP_CNT.style.height = `${size}px`;
		DRAW_PANEL.updateSize(size);
	}
</script>

<div bind:this={MAP_CNT} id="map-container" style="visibility: {currentMap ? `visible` : `hidden`};">
	<DrawPanel bind:this={DRAW_PANEL} on:map_toggle={mapDragToggler} />
	<div id="map-footer">
		<p id="coords-cnt" class=" {leafletMap && leafletMap.dragging.enabled() ? `` : `no-select`}">
			Mouse coords: <span id="coords" bind:this={currCoordsSpan}>-:-</span>
		</p>
		<div id="distance-cnt" bind:this={currDistanceDiv} />
	</div>
	<div id="map-header">
		<div id="search" class="shadow">
			Search coords
			<input type="number" name="x" class="search" maxlength="3" bind:this={SEARCH_LNG} />
			<span>:</span>
			<input type="number" name="y" class="search" maxlength="3" bind:this={SEARCH_LAT} />
			<button id="coordsSearch" bind:this={SEARCH_LATLNG_BTN}>⯐</button>
		</div>
	</div>

	<div bind:this={LEAFLET_MAP} id="map" class="crosshair-cursor" />
	<div class="leaflet-popup-tip-container  leaflet-tooltip leaflet-tooltip" style="display: none !important;" />
</div>

<style>
	#map-container * {
		box-sizing: content-box;
	}
	#map-container {
		color: #333;
		position: relative;
		width: 600px;
		height: 600px;
		max-height: var(--homepage-height);
		border-radius: 15px;
		box-sizing: content-box;
		border: 2px solid var(--accent-color);
		overflow: hidden;
	}

	#map {
		width: 100%;
		height: 100%;
		user-select: none;
		z-index: 300;
		position: relative;
	}

	#map-header {
		position: absolute;
		top: 0;
		right: 0;
		width: 400px;
		height: 40px;
		background-color: transparent;
		z-index: 600;
		display: flex;
		justify-content: end;
		align-items: center;
		padding: 0.25rem;
		overflow: hidden;
		border-radius: 5px 0 0 5px;
		user-select: none;
	}
	#map-footer {
		position: absolute;
		bottom: 1rem;
		left: 50px;
		z-index: 600;
		display: flex;
		justify-content: space-evenly;
		background-color: #3f3f3f10;
		border-radius: 15px;
	}
	:global(#distance-cnt) {
		margin-left: 2ch;
	}

	:global(#distance-cnt),
	#coords-cnt {
		font-weight: 600;
		font-size: 150%;
		padding: 0.25rem;
		text-shadow: -1px 2px 1px #5e5e5eda;
		color: var(--primary-color);
	}
	:global(#distance),
	#coords {
		color: var(--contrast-color);
		text-shadow: -1px 2px 1px var(--contrast-dark-color);
		font-weight: 700;
	}
	#coords-cnt {
		max-width: 21ch;
	}

	.crosshair-cursor {
		cursor: crosshair !important;
	}

	#search {
		border-radius: 10px;
		display: flex;
		background-color: #6200beb0;

		color: var(--primary-color);
		text-shadow: -1px 2px 1px #5e5e5eda;
		padding: 0.5rem;
		gap: 0.25rem;
		z-index: 600;
	}

	.shadow {
		box-shadow: 0 0 5px 1px black !important;
	}

	.search {
		width: 1.5rem;
		outline: none;
		-moz-appearance: textfield;
	}

	#coordsSearch {
		font-weight: 700;
		border-radius: 5px;
		border: 1px solid black;
		padding: 0.25rem;
	}

	#coordsSearch:hover {
		cursor: pointer;
		background-color: aqua;
	}

	.search::-webkit-outer-spin-button,
	.search::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}

	/*----------------------------------------------*/
	:global(.leaflet-control-container) {
		user-select: none;
	}
	.leaflet-popup-tip-container {
		display: none;
	}

	.leaflet-tooltip::before {
		display: none;
	}

	.leaflet-tooltip {
		height: 1rem;
		display: inline;
		justify-content: center;
		align-items: center;
	}
	.leaflet-container {
		background: var(--accent-dark-color);
	}
	/*----------------------------------------------*/
</style>
