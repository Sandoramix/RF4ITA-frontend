<script>
	import { createEventDispatcher, onMount } from "svelte";
	import { api, setCookie, user } from "../main";
	import { useForm, validators, HintGroup, Hint, email, required } from "svelte-use-form";

	let event = createEventDispatcher();
	export let visible = false;

	onMount(() => {
		document.querySelector(`#auth-container`).addEventListener(`click`, (ev) => {
			let target = ev.target;
			if (target.id == `auth-container`) {
				event(`exit`);
			}
		});
	});

	var usr_email, usr_password;

	let passwordError, emailError;
	passwordError = emailError = false;

	const form = useForm();

	async function login(ev) {
		ev.preventDefault();
		passwordError = emailError = false;

		let error = false;

		console.log(email, password);
		let get = await fetch(`${api}auth/login`, {
			method: "POST",
			headers: new Headers({
				"Content-Type": "application/json",
			}),
			body: JSON.stringify({ email: usr_email, password: usr_password }),
		});

		if (get.status == 404) {
			error = true;
			emailError = true;
		}
		if (get.status == 401) {
			error = true;
			passwordError = true;
		}
		if (error) return;

		let data = await get.json();
		console.log(data);
		$user = data.user;
		let token = data.user.access_token;
		token = btoa(token);
		setCookie(`token`, token, 30);
		event(`exit`);
	}
</script>

{#if visible}
	<div id="auth-container">
		<div id="auth-form">
			<h1>Login</h1>
			<form method="post" use:form on:submit={login}>
				<input
					type="email"
					name="email"
					id="email"
					use:validators={[required, email]}
					placeholder="Email"
					autocomplete="on"
					required
					class={emailError ? `red-border` : ``}
					bind:value={usr_email}
				/>
				<HintGroup for="email">
					<Hint on="required">This is a mandatory field</Hint>
					<Hint on="email" hideWhenRequired>Email is not valid</Hint>
				</HintGroup>

				<input
					type="password"
					name="password"
					id="password"
					use:validators={[required]}
					placeholder="Password"
					required
					class={passwordError ? `red-border` : ``}
					bind:value={usr_password}
				/>
				<Hint for="password" on="required">This is a mandatory field</Hint>

				<input type="submit" disabled={!$form.valid} value="Login" />
			</form>
		</div>
	</div>
{/if}

<style>
	:global(.touched:invalid) {
		border-color: red;
		outline-color: red;
	}
	#auth-container {
		position: absolute;
		width: 100vw;
		height: 100vh;
		background-color: #00000069;
		z-index: 500;
		display: flex;
		justify-content: center;
		align-items: center;
		color: var(--primary-color);
	}

	#auth-form {
		position: relative;
		display: flex;
		flex-direction: column;
		text-align: center;
		justify-content: center;
		align-items: center;
		width: 40vw;
		height: 20vh;
		min-width: 300px;
		min-height: 300px;
		background-color: var(--accent-color);
		border-radius: 15px;
		padding: 1rem 2rem;
		gap: 0.5rem;
	}

	#auth-container > * {
		margin: 0.5rem;
	}
	#auth-form input {
		width: 100%;
		height: 2rem;
		margin-top: 0.5rem;
		border: 1px solid black;
		background-color: white;
	}

	#auth-form input[type="submit"] {
		border-radius: 5px;
		width: 50%;
		height: 3rem;
	}

	#auth-form input[type="submit"]:hover {
		background-color: #d4d4d4;
		color: var(--contrast-color);
		cursor: pointer;
	}
	#auth-form input[type="submit"]:active {
		background-color: #000;
		color: var(--primary-color);
	}
</style>
