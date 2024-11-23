<script>
	let name = $state('');
	let greetMsg = $state('');
</script>

<main class="container">
	<h1
		class="font-display anim-gradient bg-secondary bg-clip-text pb-3 text-center text-4xl font-bold group-hover:text-transparent md:text-5xl"
	>
		LLamie
	</h1>

	<div class="mx-auto flex flex-col justify-center">
		<div class="max-w-xs text-center text-lg font-bold lg:max-w-3xl">
			Call some stuff from a binary
		</div>

		<form class="row">
			<input id="greet-input" placeholder="Enter a name..." bind:value={name} />
			<button type="submit">Call binary</button>
		</form>
		<p>{greetMsg}</p>
	</div>
</main>
