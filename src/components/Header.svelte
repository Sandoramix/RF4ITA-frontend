<script>
	import Donate from "./Donate.svelte";
	import IgClock from "./IGClock.svelte";

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

	function onLanguageClick() {
		if (currentLanguage.value == `ITA`) {
			event("change_language", `ENG`);
		} else {
			event("change_language", `ITA`);
		}
	}

	export let currentLanguage;
	export let mapList = [];
</script>

<header>
	<div class="clock">
		<h4>{currentLanguage.ig_clock}</h4>
		<IgClock />
	</div>

	<h1 class="header-center-cnt">
		<div class="center-item spam discord">
			<p>Community's</p>
			<a class="link" target="_blank" href="https://discord.gg/luke4316">Discord</a>
		</div>
		<div class="center-item">
			<a href="https://rf4game.com/" class="rf4-title link" target="_blank"><h1 class="rf4-title">Russian Fishing 4</h1></a>
			<span class="lang" on:click={onLanguageClick}
				><img
					style="object-fit:contain;width:20px"
					src={currentLanguage.flag_link}
					alt={currentLanguage.value}
				/>{currentLanguage.value}</span
			>
		</div>
		<div class="center-item center-item-row" style="margin-left: .5rem">
			<div class="spam author">
				<p>Made by</p>
				<a class="link" target="_blank" href="https://ko-fi.com/sandoramix">@Sandoramix</a>
			</div>
			<div class="donate-cnt">
				<Donate />
			</div>
		</div>
	</h1>

	<h1 class="maps-toggler text-center active" on:click={sidebarHandler}>{currentLanguage.maps}</h1>
	<div class="maps-select-container">
		<select bind:this={SELECT_MAP} name="select" class="maps-select" on:change={onItemClicked}>
			<option value="null" selected>{currentLanguage.choose_map}</option>
			{#each mapList as map}
				<option value={map.name}>
					{map.default_name} [ {map.unlocked_at}+ ]
				</option>
			{/each}
		</select>
	</div>
</header>

<style>
	.lang {
		cursor: pointer;
	}
	.lang:hover {
		color: var(--contrast-color);
	}

	.donate-cnt {
		height: var(--header-height);
		display: flex;
		flex-direction: column;
	}

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
		display: flex;
		flex-direction: column;
		font-weight: bold;
		color: var(--orange-light);
	}
	.center-item .rf4-title.link {
		font-size: 1.5rem;
	}

	.header-center-cnt {
		width: 75%;
		max-width: 80%;
		max-height: var(--header-height);
		display: inline-flex;
		text-align: center;
		font-size: 1.1rem;
		justify-content: center;
	}

	.header-center-cnt > :first-child {
		border-left: 1px solid var(--accent-color);
	}
	.header-center-cnt > :last-child {
		border-right: 1px solid var(--accent-color);
		height: var(--header-height);
	}

	.center-item {
		font-weight: 500;
		display: flex;
		flex-direction: column;
		width: 33%;
		align-items: center;
		justify-content: center;
	}
	.center-item-row {
		flex-direction: row;
		white-space: normal;
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
		min-width: 170px;
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
	@media screen and (max-width: 730px) {
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
		.center-item .rf4-title.link {
			font-size: 1.1rem;
		}
		.header-center-cnt {
			font-size: 0.9rem;
		}
		.link {
			font-size: 0.7rem;
		}
	}
	@media screen and (max-width: 550px) {
		.header-center-cnt {
			justify-content: end;
		}
		.spam {
			display: none !important;
		}
		.donate-cnt {
			margin-left: 1.5rem;
			height: calc(var(--header-height) * 0.75);
		}
	}
	@media screen and (max-width: 670px) {
		.spam.author {
			display: none !important;
		}
	}
	@media screen and (max-width: 610px) {
		.spam.discord {
			display: none !important;
		}
		.header-center-cnt {
			border-left: 1px solid var(--accent-color);
		}
	}
</style>
