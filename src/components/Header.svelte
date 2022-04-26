<script lang="ts">
	import IgClock from "./IGClock.svelte";

	import { createEventDispatcher } from "svelte";

	const dispatch = createEventDispatcher();

	const sidebarHandler = () => {
		dispatch(`sidebarToggle`);
	};
</script>

<header>
	<div id="clock">
		<h4>Game clock</h4>
		<IgClock />
	</div>
	<h1 id="title"><a href="/#" style="text-decoration: none;color: var(--primary-color);">Russian Fishing 4 ITA</a></h1>
	<div>
		<svg id="sidebar-toggler" viewBox="0 0 16 16" on:click={sidebarHandler}>
			<path
				d="M10.5 1a.5.5 0 0 1 .5.5v4a.5.5 0 0 1-1 0V4H1.5a.5.5 0 0 1 0-1H10V1.5a.5.5 0 0 1 .5-.5ZM12 3.5a.5.5 0 0 1 .5-.5h2a.5.5 0 0 1 0 1h-2a.5.5 0 0 1-.5-.5Zm-6.5 2A.5.5 0 0 1 6 6v1.5h8.5a.5.5 0 0 1 0 1H6V10a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5ZM1 8a.5.5 0 0 1 .5-.5h2a.5.5 0 0 1 0 1h-2A.5.5 0 0 1 1 8Zm9.5 2a.5.5 0 0 1 .5.5v4a.5.5 0 0 1-1 0V13H1.5a.5.5 0 0 1 0-1H10v-1.5a.5.5 0 0 1 .5-.5Zm1.5 2.5a.5.5 0 0 1 .5-.5h2a.5.5 0 0 1 0 1h-2a.5.5 0 0 1-.5-.5Z"
			/>
		</svg>
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
	}
	#title {
		width: 75%;
	}
	#clock {
		display: flex;
		flex-direction: column;
		text-align: center;
		align-items: center;
	}
	#sidebar-toggler {
		height: calc(var(--header-height) * 0.75);
		fill: white;
		justify-self: end;
		align-self: center;
		cursor: pointer;
	}
	#sidebar-toggler:hover {
		fill: var(--contrast-color);
	}
</style>
