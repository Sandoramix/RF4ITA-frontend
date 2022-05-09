<script>
	export let map_trophies = [];
	var input;
	export var map_trophies_filtered = map_trophies;

	const filter_fishes = () => {
		let name = input.value.toLowerCase() || "";

		map_trophies_filtered = map_trophies.filter((f) => f.fish_name.toLowerCase().includes(name));
	};
	$: visible = map_trophies.length !== 0;
	export const clearInput = () => {
		input.value = ``;
	};
</script>

{#if visible}
	<div class="container">
		<h2>Map Trophies</h2>
		<input type="text" class="search_fish" passive:true bind:this={input} on:input={filter_fishes} placeholder="Search for fish" />

		<li id="trophies">
			{#each map_trophies_filtered as fish}
				<li class="item">
					<span class="fish">{fish.fish_name}</span>
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
		</li>
	</div>
{/if}

<style>
	h2 {
		padding: 0.25rem;
		background-color: #1a0035;
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

		text-align: center;
	}
	#trophies {
		margin: 0;
		padding: 0;
		width: 100%;
		user-select: none;
		list-style: none;

		display: flex;
		flex-direction: column;

		align-items: center;
		text-align: center;
		justify-content: start;
		font-size: 19px;
		height: 100%;
		max-height: 30vh;
		overflow-y: auto;
	}
	.item {
		width: 100%;
		border-top: 1px solid var(--primary-color);
		display: flex;

		justify-content: space-between;
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
