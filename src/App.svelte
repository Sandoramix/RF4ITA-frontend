<script>
	import Adminpanel from "./components/Adminpanel.svelte";
	import { api, user, getCookie, setCookie } from "./main";

	import Header from "./components/Header.svelte";

	import { onMount } from "svelte";
	import Authme from "./components/Authme.svelte";

	import GameMap from "./components/GameMap.svelte";
	import Sidebar from "./components/Sidebar.svelte";

	$: loggedIn = $user.access_token != null;

	var gameMap,
		editorMode = true,
		isAuthenticating = false,
		sidebarActive = false,
		currentMapName = "",
		mapList;

	mapList = [];

	async function loginFromCookie() {
		let _tk = getCookie(`token`) || false;
		let get = await fetch(`${api}auth/login`, {
			method: "POST",
			headers: new Headers({
				Authorization: `Bearer ${_tk}`,
			}),
		});
		let data = await get.json();
		let res = data.user;
		if (res) {
			$user.access_token = res.access_token;
		} else {
			$user = {
				access_token: null,
				email: null,
				id: null,
			};
		}
	}
	function logout() {
		$user = {
			access_token: null,
			email: null,
			id: null,
		};
		document.cookie = "token=; Path=/; Expires=Thu, 01 Jan 1970 00:00:01 GMT;";
	}

	//INITIAL LOAD
	onMount(async () => {
		let get = await fetch(`${api}maps/`);
		let data = await get.json();

		mapList = data.results;

		console.log(mapList);
		await loginFromCookie();
		init();

		//TODO REMOVE
		updateCurrentMap({ detail: `thecottagepond` });
	});

	//SYNC INITIAL LOAD
	const init = () => {};

	const updateCurrentMap = (ev) => {
		let find = mapList?.find((m) => m.name === ev.detail);
		let currentMap = find;
		currentMapName = currentMap.formatted_name;
		gameMap.updateMap(currentMap);
	};

	const sidebarToggleHandler = () => {
		sidebarActive = !sidebarActive;
		document.documentElement.style.setProperty(`--sidebar-width`, `${sidebarActive ? `${200}px` : "0px"}`);
	};

	setTimeout(sidebarToggleHandler, 200);
</script>

<main>
	{#if isAuthenticating}
		<Authme visible={isAuthenticating} on:exit={() => (isAuthenticating = false)} />
	{/if}

	<Header on:sidebarToggle={sidebarToggleHandler} />

	<div id="page">
		<div class="admin-panel">
			{#if loggedIn}
				<Adminpanel />
			{/if}
		</div>

		<div id="gmap">
			<GameMap bind:this={gameMap} {editorMode} />
		</div>
	</div>

	<Sidebar
		visible={sidebarActive}
		{mapList}
		{loggedIn}
		on:change_map={updateCurrentMap}
		on:login_click={() => (isAuthenticating = true)}
		on:logout_click={logout}
	/>
</main>

<style>
	main {
		height: 100%;
	}
	#page {
		width: calc(100% - var(--sidebar-width));
		height: calc(100vh - var(--header-height));
		overflow-y: auto;

		display: flex;

		align-items: center;
	}

	.admin-panel {
		width: 25% !important;

		height: 100%;
		justify-self: start;
	}
	#gmap {
		width: 80%;
	}
</style>
