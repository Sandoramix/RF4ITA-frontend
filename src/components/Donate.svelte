<script>
	// import { onMount } from "svelte";

	// onMount(() => {
	// 	let cachedNextPopup = localStorage.getItem("nextPopup");
	// 	if (!cachedNextPopup) {
	// 		setTimeout(() => {
	// 			popMessage();
	// 		}, 600000);
	// 		return;
	// 	}

	// 	let nextView = new Date(cachedNextPopup);

	// 	let now = new Date();
	// 	if (now >= nextView) {
	// 		console.log(`ciao`);
	// 		popMessage();
	// 	}
	// 	popMessage();
	// });
	// let timer = null;

	// async function handleResponse() {}

	// async function popMessage() {}
</script>

<a href="https://streamlabs.com/sandoramix/tip" target="_blank"><img class="donate-img" src="./images/donate.png" alt="donate" /> </a>

<!-- <div class="donate-popup-cnt">
	<div class="donate-popup" />
</div> -->
<style>
	/* .donate-popup-cnt {
		position: absolute;
		height: 5rem;
		width: 10rem;
		bottom: -5rem;
		left: -2.5rem;
		background-color: rosybrown;
	}
	.donate-popup-cnt::before {
		content: "";
		width: 0;
		height: 0;
		border-left: 16px solid transparent;
		border-right: 16px solid transparent;
		border-bottom: 15px solid red;

		position: absolute;
		top: -15px;
		left: 50%;
		transform: translateX(-50%);
	} */
	:global(.donate-popup-cnt.hidden) {
		visibility: hidden;
		height: 0;
	}

	a {
		height: 100%;
	}
	.donate-img {
		height: 100%;
		width: auto;
		object-fit: contain;
	}
</style>
