<script>
	import Policy from "./routes/Policy.svelte";
	import Home from "./routes/Home.svelte";
	import { api } from "./extra";
	import Router from "svelte-spa-router";

	//INITIAL LOAD
	fetch(`${api}ismobile`)
		.then((res) => res.json())
		.then((res) => {
			if (res.result) {
				isMobile = true;
				alert(
					`This site is meant to be used on computer browsers.\nTo have an better experience we recommend you to not use it from mobile phones`,
				);
			}
		});
	const routes = {
		"/policy": Policy,
		"*": Home,
	};
</script>

<Router {routes} />

<style>
	/* main {
		height: calc(100% - var(--header-height));
		position: relative;
	} */
</style>
