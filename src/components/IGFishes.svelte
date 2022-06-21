<script>
	import { createEventDispatcher } from "svelte";

	var event = createEventDispatcher();

	export var allFishes = [];
	export var mapFishes = [];
	export var mapTrophies = [];

	$: visible = currentlyShowingFishes.length !== 0;
	var input;

	$: currentlyShowingFishes = mapTrophiesToggler ? mapTrophies : mapFishesToggler ? mapFishes : allFishes;

	$: currentlyFilteredShowingFishes = currentlyShowingFishes.filter((fish) =>
		fish.fish_name.toLowerCase().includes(input ? input.toLowerCase() : ``),
	);

	var mapTrophiesToggler = false;
	var mapFishesToggler = true;

	// //EVENTS

	function onFocusToggle(flag) {
		event("focus_toggle", flag);
	}
</script>

{#if visible}
	<div class="container">
		<div class="header">
			<h3>Fishes</h3>
			<input on:focus={() => onFocusToggle(true)} on:blur={() => onFocusToggle(false)} type="text" class="search_fish" bind:value={input} />

			<div class="filters">
				<label class="filter {mapTrophies.length == 0 ? `disabled` : ``}" value="trophy" for="trophy_toggler"
					><hr class="line" />
					<input
						type="checkbox"
						disabled={mapTrophies.length == 0}
						bind:checked={mapTrophiesToggler}
						id="trophy_toggler"
						name="filter"
					/>Map trophies</label
				>

				<label class="filter {mapTrophiesToggler ? `disabled` : ``}" value="fish" for="fish_toggler"
					><hr class="line" />
					<input
						type="checkbox"
						disabled={mapTrophiesToggler || mapFishes.length == 0}
						bind:checked={mapFishesToggler}
						id="fish_toggler"
						name="filter"
					/>Map fish</label
				>
			</div>
		</div>
		<ul id="fishes">
			{#each currentlyFilteredShowingFishes as fish}
				<li class="item">
					<span class="fish">{fish.default_name}</span>
					<div class="fish-sub">
						<div>
							<img src="images/trophy.png" alt="Trophy" />
							<span
								>{fish.fish_trophy != null
									? fish.fish_trophy < 1000
										? `${fish.fish_trophy}gr`
										: `${fish.fish_trophy / 1000}kg`
									: `?`}
							</span>
						</div>
						<div>
							<img src="images/super-trophy.png" alt="Trophy" />
							<span>
								{fish.fish_super_trophy != null
									? fish.fish_super_trophy < 1000
										? `${fish.fish_super_trophy}gr`
										: `${fish.fish_super_trophy / 1000}kg`
									: `?`}
							</span>
						</div>
					</div>
				</li>
			{/each}
		</ul>
	</div>
{/if}

<style>
	.header {
		border-bottom: solid 1px var(--purple-outline-color);
		padding: 0.25rem;
	}
	.filters {
		display: flex;
		background-color: #0e001d;
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
		color: gray;
	}
	.filter.disabled:hover {
		cursor: not-allowed !important;
		color: gainsboro;
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
	}

	#fishes {
		width: 100%;
		user-select: none;
		list-style: none;

		display: flex;
		flex-direction: column;
		text-align: center;
		justify-content: start;
		font-size: 19px;
		height: inherit;
		max-height: 30vh;

		overflow-y: auto;
	}
	.item {
		width: inherit;
		border-top: 1px solid var(--primary-color);
		display: flex;

		justify-content: start;
	}
	.item:nth-child(odd) {
		background-color: #000;
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
		/* color: rgb(0, 159, 233); */
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
	}
	.fish-sub img {
		width: 25%;
		aspect-ratio: 1;
		object-fit: scale-down;
	}
</style>
