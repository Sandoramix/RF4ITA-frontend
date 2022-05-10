<script lang="ts">
	import IgClock from ".\\IGClock.svelte";

	import { createEventDispatcher, onMount } from "svelte";

	const event = createEventDispatcher();

	//ELEMENTS
	var SELECT_MAP;

	const sidebarHandler = (ev) => {
		ev.target.classList.toggle(`active`);
		event(`sidebarToggle`);
	};

	function onItemClicked() {
		let name = SELECT_MAP.selectedOptions[0].value;
		event(`change_map`, name);
	}

	export function manualSelect(name) {
		if (SELECT_MAP) {
			[...SELECT_MAP.children].forEach((opt) => {
				opt.selected = opt.value === name;
			});
		}
	}

	export let mapList = [];
</script>

<header>
	<div class="clock">
		<h4>IG Clock</h4>
		<IgClock />
	</div>

	<h1 class="center">
		<div class="spam">
			<p id="discord">Community's</p>
			<a class="link" target="_blank" href="https://discord.gg/luke4316">Discord</a>
		</div>
		<div class="rf4-title"><a href="https://rf4game.com/" class="link rf4-title" target="_blank">Russian Fishing 4 </a>ITA</div>
		<div class="spam">
			<p id="author">Made by</p>
			<a class="link" target="_blank" href="https://www.instagram.com/sandoramixer/">@Sandoramix</a>
		</div>
	</h1>

	<h1 class="maps-toggler text-center active" on:click={sidebarHandler}>Maps</h1>
	<div class="maps-select-container">
		<select bind:this={SELECT_MAP} name="select" class="maps-select" on:change={onItemClicked}>
			<option value="null" selected>Choose The Map</option>
			{#each mapList as map}
				<option value={map.name}>
					{map.formatted_name}
				</option>
			{/each}
		</select>
	</div>
</header>

<style>
	header {
		width: 100%;
		height: var(--header-height);
		display: flex;
		justify-content: space-around;
		background-color: var(--header-bg);
		text-align: center;
		align-items: center;
		color: white;
		user-select: none;
		white-space: pre;
		overflow-x: auto;
		overflow-y: hidden;
	}

	/* LINKS */
	.link {
		text-decoration: dotted;
		color: var(--orange);
		font-size: medium;
		font-size: 0.9rem;
		font-weight: 700;
	}
	.link:hover {
		color: var(--contrast-color);
	}

	/* HEADER INFOS */
	.rf4-title {
		font-size: 1.5rem;
		display: inline;
		font-weight: bold;
		color: var(--orange-light);
	}
	.rf4-title .link {
		font-size: 1.5rem;
	}

	.center {
		width: 75%;
		max-width: 80%;
		max-height: var(--header-height);
		display: inline-flex;
		text-align: center;
		font-size: 1.1rem;
		justify-content: center;
	}

	.center > :first-child {
		border-left: 1px solid var(--accent-color);
	}
	.center > :last-child {
		border-right: 1px solid var(--accent-color);
		height: var(--header-height);
	}

	.center > div {
		font-weight: 500;
		display: flex;
		flex-direction: column;
		width: 33%;
		align-items: center;
		justify-content: center;
	}

	.clock {
		min-width: 90px;
		max-width: 10%;
		font-size: 1rem;
		display: flex;
		flex-direction: column;
		text-align: center;
		align-items: center;
	}

	/* MAPS TOGGLER / LIST */
	.maps-toggler,
	.maps-select-container {
		width: 10%;
		height: var(--header-height);
		line-height: var(--header-height);
		min-width: fit-content;
		justify-self: end;
		align-self: center;
	}
	.maps-toggler {
		display: block;

		color: white;

		cursor: pointer;
	}
	.maps-toggler.active {
		color: var(--contrast-color);
	}

	.maps-toggler:hover {
		color: var(--contrast-dark-color);
	}

	.maps-select-container {
		display: none;
	}

	.maps-select {
		text-align: center;
		width: 100%;
		height: 70%;
		font-size: 0.8rem;
		background-color: transparent;
		border: 1px solid var(--contrast-color);
		color: var(--primary-color);
		border-radius: 5px;
		cursor: pointer;
	}
	.maps-select option {
		background-color: var(--accent-dark-color);
		cursor: pointer;
	}

	.maps-select option:checked {
		background-color: var(--contrast-color);
		color: var(--accent-dark-color);
		font-weight: bold;
	}

	/* ----------------------------- */

	/* MOBILE DEVICES */
	@media screen and (max-width: 700px) {
		.maps-toggler {
			display: none;
		}
		.maps-select-container {
			display: block;
		}
		.clock {
			max-width: 5%;
			min-width: 60px;
			font-size: 0.8rem;
		}
		.rf4-title {
			font-size: 1.1rem;
		}
		.rf4-title .link {
			font-size: 1.1rem;
		}
		.center {
			font-size: 0.9rem;
		}
		.link {
			font-size: 0.7rem;
		}
	}
	@media screen and (max-width: 550px) {
		.spam {
			display: none !important;
		}
	}
</style>
