<script>
	import Home from "./routes/Home.svelte";
	import Router from "svelte-spa-router";

	const routes = {
		"*": Home,
	};
</script>

<Router {routes} />

<style>
</style>
