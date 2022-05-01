<script lang="ts">
	import IgClock from "./IGClock.svelte";

	import { createEventDispatcher } from "svelte";

	const dispatch = createEventDispatcher();

	const sidebarHandler = (ev) => {
		ev.target.classList.toggle(`active`);
		dispatch(`sidebarToggle`);
	};
</script>

<header>
	<div id="clock">
		<h4>Game clock</h4>
		<IgClock />
	</div>
	<h1 id="title">
		<div>Russian Fishing 4 ITA</div>
		<a href="https://discord.gg/luke4316" id="discord"><div>Discord</div></a>
	</h1>

	<h1 id="sidebar-toggler" class="text-center active" on:click={sidebarHandler}>Maps</h1>
</header>

<style>
	#discord {
		text-decoration: underline;
		color: var(--primary-color);
		font-size: medium;
	}
	#discord:hover {
		color: var(--contrast-color);
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
	}
	#title {
		width: 75%;
		max-width: 80%;
		max-height: var(--header-height);
	}
	#clock {
		min-width: 90px;
		max-width: 10%;
		display: flex;
		flex-direction: column;
		text-align: center;
		align-items: center;
	}
	#sidebar-toggler {
		height: var(--header-height);
		line-height: var(--header-height);
		width: 10%;
		min-width: fit-content;
		color: white;
		justify-self: end;
		align-self: center;
		cursor: pointer;
	}
	#sidebar-toggler.active {
		color: var(--contrast-color);
	}

	#sidebar-toggler:hover {
		color: var(--contrast-dark-color);
	}
</style>
