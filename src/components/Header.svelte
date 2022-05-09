<script lang="ts">
	import IgClock from ".\\IGClock.svelte";

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
		<div>
			<p id="discord">Community's</p>
			<a class="link" target="_blank" href="https://discord.gg/luke4316">Discord</a>
		</div>
		<div><p id="rf4-title">Russian Fishing 4 ITA</p></div>
		<div>
			<p id="author">Made by</p>
			<a class="link" target="_blank" href="https://www.instagram.com/sandoramixer/">@Sandoramix</a>
		</div>
	</h1>

	<h1 id="sidebar-toggler" class="text-center active" on:click={sidebarHandler}>Maps</h1>
</header>

<style>
	.link {
		text-decoration: dotted;
		color: var(--orange);
		font-size: medium;
		font-size: 13px;
		font-weight: 700;
	}
	.link:hover {
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
	#rf4-title {
		font-size: 24px;
		display: inline;
		font-weight: bold;
		color: var(--orange-light);
	}

	#title {
		width: 75%;
		max-width: 80%;
		max-height: var(--header-height);
		display: inline-flex;
		text-align: center;

		justify-content: center;
	}

	#title > :first-child {
		border-left: 1px solid var(--accent-color);
	}
	#title > :last-child {
		border-right: 1px solid var(--accent-color);
		height: var(--header-height);
	}

	#title > div {
		font-weight: 500;
		display: flex;
		flex-direction: column;
		width: 33%;
		font-size: 18px;
		align-items: center;
		justify-content: center;
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
