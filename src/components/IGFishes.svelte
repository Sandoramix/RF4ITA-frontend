<script>
	import { createEventDispatcher, onMount } from "svelte";

	var event = createEventDispatcher();

	export var currentLanguage;
	export var allFishes = [];
	export var mapFishes = [];
	export var mapTrophies = [];

	var visible = false;
	var input;

	$: currentlyShowingFishes = mapT && mapTrophies.length !== 0 ? mapTrophies : mapF && mapFishes.length !== 0 ? mapFishes : allFishes;
	$: currentlyFilteredShowingFishes = currentlyShowingFishes.filter((fish) =>
		currentLanguage.value === "ITA"
			? fish.name_it.toLowerCase().includes(input ? input.toLowerCase() : ``)
			: fish.name.toLowerCase().includes(input ? input.toLowerCase() : ``),
	);
	var mapT = false;
	$: mapTAvaiable = mapTrophies.length !== 0;

	var mapF = true;
	$: mapFAviable = mapFishes.length !== 0 && !mapT;
	export const clearChecks = () => {
		mapT = false;
		mapF = false;
	};
	export const toggle = () => {
		visible = !visible;
	};
	// //EVENTS

	function onFocusToggle(flag) {
		event("focus_toggle", flag);
	}
</script>

<div class={`container ${visible ? `` : `hidden`}`}>
	<div class="header">
		<h3>{currentLanguage.fish}</h3>
		<input
			on:focus={() => onFocusToggle(true)}
			on:blur={() => onFocusToggle(false)}
			type="text"
			class="search_fish"
			bind:value={input}
			placeholder={currentLanguage.search_fish}
		/>

		<div class="filters">
			<label class="filter {!mapTAvaiable ? `disabled` : ``}" value="trophy" for="trophy_toggler"
				><hr class="line" />
				<input
					class={`${mapT}`}
					type="checkbox"
					disabled={!mapTAvaiable}
					bind:checked={mapT}
					id="trophy_toggler"
					name="filter"
				/>{currentLanguage.this_map_trophies}</label
			>

			<label class="filter {(mapT && mapTrophies.length !== 0) || mapFishes.length === 0 ? `disabled` : ``}" value="fish" for="fish_toggler"
				><hr class="line" />
				<input
					type="checkbox"
					disabled={(mapT && mapTrophies.length !== 0) || mapFishes.length === 0}
					bind:checked={mapF}
					id="fish_toggler"
					name="filter"
				/>{currentLanguage.this_map_fish}</label
			>
		</div>
	</div>
	<ul id="fishes">
		{#each currentlyFilteredShowingFishes as fish}
			<li class="item">
				<span class="fish">{currentLanguage.value == "ITA" ? fish.name_it : fish.name}</span>
				<div class="fish-sub">
					<div>
						<img src="images/trophy.png" alt="Trophy" />
						<span>{fish.trophy != null ? (fish.trophy < 1000 ? `${fish.trophy}gr` : `${fish.trophy / 1000}kg`) : `?`} </span>
					</div>
					<div>
						<img src="images/super-trophy.png" alt="Trophy" />
						<span>
							{fish.super_trophy != null
								? fish.super_trophy < 1000
									? `${fish.super_trophy}gr`
									: `${fish.super_trophy / 1000}kg`
								: `?`}
						</span>
					</div>
				</div>
			</li>
		{/each}
	</ul>
	<div
		class="toggler"
		passive:true
		on:click={() => {
			visible = !visible;
		}}
	>
		F<span style="color:whitesmoke;" />
	</div>
</div>

<style>
	.toggler {
		position: absolute;
		width: 3rem;
		height: 7rem;
		top: 0;
		left: calc(var(--map-left-panel-width) - 1px);

		display: flex;
		justify-content: center;
		align-items: center;
		background-color: var(--red-dark-color);

		cursor: pointer;
		padding: 0.5rem;
		text-align: center;

		border-top-right-radius: 5px;
		border-bottom-right-radius: 5px;
		user-select: none;
		font-weight: bold;

		border-top: 1px solid var(--outline-color);
		border-bottom: 1px solid var(--outline-color);
		border-right: 1px solid var(--outline-color);

		transition: left 0.5s ease-in-out;
		z-index: 65;
	}
	.header {
		position: relative;
		display: flex;
		flex-direction: column;
		justify-content: center;
		gap: 0.25em;
		height: 7rem;
		border-bottom: solid 1px var(--orange);
		padding: 0.25rem;
		background-color: #07001b;
	}
	.filters {
		display: flex;
		align-items: center;
		justify-content: space-around;
	}
	.filter {
		position: relative;
		font-size: small;

		padding: 0.125rem;
		border-radius: 5px;
	}
	.filter input,
	.filter:hover {
		cursor: pointer;
		color: var(--contrast-color);
	}
	.filter .line {
		border-color: red;
		display: none;
		position: absolute;
		width: 100%;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}
	.filter.disabled .filter input,
	.filter.disabled {
		color: #808080;
	}
	.filter.disabled:hover {
		cursor: not-allowed !important;
		color: #585858;
	}
	.filter.disabled .line {
		display: block;
	}

	.search_fish {
		width: 99%;
		margin: 0.125rem;
	}
	h3 {
		padding: 0.25rem;
	}
	.container::-webkit-scrollbar-thumb,
	.container *::-webkit-scrollbar-thumb {
		background-color: var(--accent-color);
	}
	.container::-webkit-scrollbar-track,
	.container *::-webkit-scrollbar-track {
		width: 5px;
	}
	.container {
		background-color: var(--accent-dark-color);
		width: var(--map-left-panel-width);
		border-bottom-right-radius: 10px;
		text-align: center;
		border-top: 1px solid var(--outline-color);
		border-bottom: 1px solid var(--outline-color);
		border-right: 1px solid var(--outline-color);
		position: fixed;
		top: 30%;
		height: 51vh;
		z-index: 60;
		transition: transform 0.3s ease-in-out;
	}
	.container.hidden {
		transform: translateX(-100%);
	}

	#fishes {
		height: calc(50vh - 7rem);
		max-height: calc(51vh - 7rem);
		width: 100%;
		user-select: none;
		list-style: none;

		display: flex;
		flex-direction: column;
		text-align: center;
		justify-content: start;
		font-size: 19px;
		height: inherit;

		overflow-y: auto;
	}
	.item {
		width: inherit;
		border-bottom: 1px solid var(--primary-color);
		display: flex;
		align-items: center;
		justify-content: start;
	}
	.item:nth-child(odd) {
		background-color: rgb(13, 0, 19);
	}

	.fish {
		font-weight: bold;
		width: 70%;
		font-size: 19px;
	}
	.fish-sub {
		border-left: 1px solid var(--primary-color);
		display: flex;
		flex-direction: column;
		justify-self: end;
		gap: 0.125rem;
		justify-content: center;
		font-size: 16px;
		text-align: end;
		color: var(--contrast-color);
		width: 30%;

		font-weight: 400;
	}
	.fish-sub > div {
		height: 50%;
		display: flex;
		justify-content: space-between;
	}
	.fish-sub > div:first-child {
		border-bottom: thin solid #f5f5f580;
	}

	.fish-sub span {
		width: 75%;
		user-select: text;
		margin-right: 10%;
		font-weight: bold;
	}
	.fish-sub img {
		width: 25%;
		aspect-ratio: 1;
		object-fit: scale-down;
	}
</style>
