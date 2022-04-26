<script lang="ts">
	import { createEventDispatcher } from "svelte";

	const event = createEventDispatcher();

	export let visible = true;
	export let mapList = [];
	export let loggedIn = false;

	const onItemClicked = (newMap) => {
		currentMap = newMap;
		event(`change_map`, newMap.name);
		return;
	};

	let currentMap = null;
</script>

<aside class={visible ? `` : `hidden`}>
	<h1 class="text-center">Maps</h1>

	<div class="category-list">
		{#each mapList as map}
			<div
				class="category-list__item {currentMap == map ? `active` : ``}"
				on:click={() => {
					onItemClicked(map);
				}}
			>
				{map.formatted_name}
			</div>
		{/each}
	</div>

	<div id="footer">
		{#if loggedIn}
			<button on:click={() => event(`logout_click`)}>Logout</button>
		{:else}
			<button on:click={() => event(`login_click`)}>Login</button>
		{/if}
	</div>
</aside>

<style>
	#footer {
		display: flex;
		flex-direction: column;
		margin-top: auto;

		padding: 0.5rem;
		gap: 0.5rem;
		justify-content: center;
		align-items: center;
		background-color: #111111;
	}
	#footer button {
		width: 80%;
		height: 2.5rem;
		border-radius: 5px;
		background-color: transparent;
		color: white;
		border: 1px solid white;
	}
	#footer button:hover {
		background-color: var(--contrast-color);
		color: black;
		cursor: pointer;
	}

	aside {
		user-select: none;
		color: white;
		top: var(--header-height);
		position: fixed;
		right: 0;
		height: calc(100% - var(--header-height));

		display: flex;
		flex-direction: column;
		justify-content: start;
		width: var(--sidebar-width);
		background-color: var(--sidebar-bg);

		gap: 0.5rem;
		overflow-y: auto;
		overflow-x: hidden;
		transform: translateX(0);
		transition: transform 0.5s ease-in-out, width 0.5s ease-in-out;
		box-shadow: 0 0 10px 5px #000000;

		clip-path: inset(0px 0px 0px -10px);
	}
	aside::-webkit-scrollbar-thumb,
	aside *::-webkit-scrollbar-thumb {
		background-color: var(--contrast-color);
	}
	aside::-webkit-scrollbar-track,
	aside *::-webkit-scrollbar-track {
		width: 5px;
	}

	aside.hidden {
		transform: translateX(150%);
		box-shadow: none;
	}

	.category-list {
		overflow-x: hidden;
		display: flex;
		flex-direction: column;

		width: 100%;

		transition: height 0.5s ease-in-out, border 0.5s ease-in-out;
	}

	.category-list__item {
		background-color: transparent;

		text-align: center;

		border-top: solid 1px var(--primary-color);
		padding: 0.5rem 1rem;
	}
	.category-list__item:last-child {
		border-bottom: solid 1px var(--primary-color);
	}

	.category-list__item:hover {
		background-color: var(--accent-color);
	}

	.category-title:hover,
	.category-list__item:hover {
		cursor: pointer;
	}

	.category-list__item.active {
		background-color: var(--accent-color);
	}
</style>
