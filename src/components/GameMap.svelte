<script>
	import DrawPanel from "./DrawPanel.svelte";
	import L from "leaflet";
	import { onMount } from "svelte";
	import { api } from "../extra";

	var currentMap;

	var SEARCH_LATLNG_BTN,
		SEARCH_LNG,
		SEARCH_LAT,
		MAP_CNT,
		LEAFLET_MAP,
		COORDS_CONTENT,
		IMG_OUTER_SIZE = 1024,
		IMG_MIN_INNER_PX = 40,
		IMG_MAX_INNER_PX = 986,
		OUTER_BOUNDS = [
			[0, 0],
			[IMG_OUTER_SIZE, IMG_OUTER_SIZE],
		],
		INNER_BOUNDS = [
			[IMG_MIN_INNER_PX, IMG_MIN_INNER_PX],
			[IMG_MAX_INNER_PX, IMG_MAX_INNER_PX],
		],
		IMG_INNER_SIZE = IMG_OUTER_SIZE - IMG_MIN_INNER_PX * 2,
		IG_MAP_SQUARE_LENGTH = 94;

	var layerObject = document.createElement(`object`),
		leafletMap,
		currentCoordinateCircle = L.circleMarker(),
		mapTileLayer = L.tileLayer(),
		layerGroup,
		xRatio = 0,
		yRatio = 0,
		x = 0,
		y = 0,
		userFirstMark = L.circleMarker(),
		userSecondMark = L.circleMarker(),
		userMarksLine = L.polyline([
			[-1, -1],
			[-1, -1],
		]),
		polylineTooltip = L.tooltip();

	userFirstMark = userSecondMark = userMarksLine = polylineTooltip = null;

	onMount(() => {
		SEARCH_LATLNG_BTN = document.querySelector(`#coordsSearch`);
		SEARCH_LNG = document.querySelector(`#search-x`);
		SEARCH_LAT = document.querySelector(`#search-y`);
		MAP_CNT = document.querySelector(`#map-container`);
		LEAFLET_MAP = document.querySelector(`#map`);
		COORDS_CONTENT = document.querySelector(`#coords`);
		SEARCH_LATLNG_BTN.addEventListener(`click`, () => searchCoordsHandler(true));
	});

	export async function updateMap(map) {
		currentMap = map;
		if (!map || !currentMap) return;

		xRatio = IMG_INNER_SIZE / (currentMap.limits.x.max - currentMap.limits.x.min);
		yRatio = IMG_INNER_SIZE / (currentMap.limits.y.max - currentMap.limits.y.min);

		LEAFLET_MAP.innerHTML = `<div id="map"></div>`;
		if (leafletMap) {
			leafletMap.invalidateSize();
			leafletMap.off();
			leafletMap.remove();
		}

		mapTileLayer = L.tileLayer(`${api}maps/${currentMap.name}/{z}_{x}_{y}.jpg`, {
			minZoom: 0,
			maxZoom: 2,
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
		let get = await fetch(`${api}maps/${currentMap.name}/ground`);

		var allLayers = [mapTileLayer];

		if (get.status != 404) {
			obj.data = `${api}maps/${currentMap.name}/ground`;

			var grOverlay = L.svgOverlay(obj, OUTER_BOUNDS);
			allLayers.push(grOverlay);
		}

		leafletMap = L.map(`map`, {
			crs: customCRS,
			layers: allLayers,
			maxBounds: OUTER_BOUNDS,
			doubleClickZoom: false,
		}).setView([0, 0], 1);

		if (get.status != 404) {
			L.control
				.layers(
					{},
					{
						Ground: grOverlay,
					},
				)
				.setPosition("topleft")
				.addTo(leafletMap);
		}

		leafletMap.on("click", onLeafletMapClick);

		leafletMap.on("mousemove", (ev) => {
			y = IMG_OUTER_SIZE - ev.latlng.lat;
			x = ev.latlng.lng;

			let check = isCoordsInBounds(x, y);

			let igX = mapToGameCoord(x, xRatio, currentMap.limits.x);
			let igY = mapToGameCoord(y, yRatio, currentMap.limits.y);

			COORDS_CONTENT.textContent = `${check ? `${igX}:${igY}` : `-:-`}`;
			if (!check) return;
			if (userFirstMark && !userSecondMark) {
				updatePolyline(ev.latlng);
			}
		});
	}
	function onLeafletMapClick(ev) {
		y = ev.latlng.lat;
		x = ev.latlng.lng;

		if (!isCoordsInBounds(x, y)) return;

		createUserMarkerHandler(ev.latlng);
	}
	function newCircleMarker(latlng, radius = 10, tooltipContent = null, color = `#3388ff`) {
		let marker = L.circleMarker(latlng, { radius, color });
		if (tooltipContent) {
			let tip = L.tooltip({ direction: "auto", interactive: false }).setContent(tooltipContent);
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
			polylineTooltip.removeFrom(leafletMap);
			userFirstMark = userSecondMark = userMarksLine = polylineTooltip = null;
			return;
		}

		let igx = mapToGameCoord(latlng.lng, xRatio, currentMap.limits.x),
			igy = mapToGameCoord(IMG_OUTER_SIZE - latlng.lat, yRatio, currentMap.limits.y);

		if (userFirstMark) {
			userSecondMark = newCircleMarker(latlng, 8, `${igx}:${igy}`, `#40FF40`).addTo(leafletMap);
			updatePolyline(latlng);

			return;
		}
		userFirstMark = newCircleMarker(latlng, 10, `${igx}:${igy}`).addTo(leafletMap);
		polylineTooltip = L.tooltip({ permanent: true, direction: "top", sticky: true });
		polylineTooltip.setContent(`Distance: 0m`);

		userMarksLine = L.polyline([userFirstMark.getLatLng(), latlng], {}).addTo(leafletMap);
		userMarksLine.bindTooltip(polylineTooltip);
		userMarksLine.openTooltip();
	}

	function updatePolyline(latlng, offset = 0) {
		if (!userFirstMark) return;
		let midpoint = midpointBetweenLatlng(userFirstMark.getLatLng(), latlng);

		let direction = getOrientation(latlng);

		let lat = latlng.lat,
			lng = latlng.lng;
		lat = direction.includes("N") ? lat + offset : lat - offset;
		lng = direction.includes("E") ? lng - offset : lng + offset;

		userMarksLine.setLatLngs([userFirstMark.getLatLng(), L.latLng(lat, lng)]);

		let distance = distanceBetweenPoints(userFirstMark.getLatLng(), latlng, IG_MAP_SQUARE_LENGTH / currentMap.square_distance);

		polylineTooltip.setLatLng(midpoint).setContent(`Distance: <b>${distance}m</b> <em><b>${distance !== 0 ? direction : ``}</b></em>`);
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
	function midpointBetweenLatlng(latlng1, latlng2) {
		return L.latLng((latlng1.lat + latlng2.lat) / 2, (latlng1.lng + latlng2.lng) / 2);
	}

	function searchCoordInBounds(coord, limits, element) {
		if (coord > limits.max || coord < limits.min) {
			element.classList.add(`red-border`);
			return false;
		}

		element.classList.remove(`red-border`);
		return true;
	}

	const mapDragToggler = () => {
		if (leafletMap.dragging.enabled()) {
			return leafletMap.dragging.disable();
		}
		leafletMap.dragging.enable();
	};
</script>

<div id="map-container" style="visibility: {currentMap ? `visible` : `hidden`};">
	<DrawPanel on:map_toggle={mapDragToggler} />
	<div id="map-header">
		<span id="coords" class="shadow {leafletMap && leafletMap.dragging.enabled() ? `` : `no-select`}">-:-</span>
		<div id="search" class="shadow">
			<span>Coordinate search:</span>
			<input type="number" name="x" id="search-x" class="search" maxlength="3" bind:this={SEARCH_LNG} />
			<span>:</span>
			<input type="number" name="y" id="search-y" class="search" maxlength="3" bind:this={SEARCH_LAT} />
			<button id="coordsSearch">⯐</button>
		</div>
	</div>
	<div id="map" class="crosshair-cursor" />
	<div class="leaflet-popup-tip-container  leaflet-tooltip leaflet-tooltip" style="display: none !important;" />
</div>

<link rel="stylesheet" href="https://ita.russianfishing.repl.co/leaflet/leaflet.css" />

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
		width: 600px;
		height: 600px;
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

	#coords {
		text-align: center;
		min-width: 4ch;
		padding: 0.25ch;
		border-radius: 0.5rem;
		background-color: #ffffff9c;
		font-size: 150%;
		box-shadow: 0 0 1px inset darkgray;
		margin-right: 0.5rem;
	}

	.crosshair-cursor {
		cursor: crosshair !important;
	}

	#search {
		border-radius: 10px;
		display: flex;
		background-color: #ffffff9c;
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

	/*----------------------------------------------*/
</style>
