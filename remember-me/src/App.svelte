<script>
	import ThingThumbnail from "./ThingThumbnail.svelte";

	// export let name;
	let things;

	const getThings = async () => {
		return fetch('https://remember-me-rails.herokuapp.com/things')
			.then(response => response.json())
			.then(data => {
				things = data;
				return things;
			});
	};
</script>

<main>
	<nav class="level">
		<div class="level-item has-text-centered">
			<h1 class="title">Remember Me!</h1>
		</div>
	</nav>
	
	<div>
		<h1 class='title'>Things:</h1>
		<div class='is-flex is-flex-direction-row is-flex-wrap-wrap is-justify-content-center'>
			{#await getThings()}
				<p>Loading your things</p>
			{:then}
				{#each things as thing}
					<ThingThumbnail name={thing.name} id={thing.id} />
				{/each}
			{:catch error}
				<p>Error loading things: {error}</p>
			{/await}
		</div>
	</div>
</main>

<style>
</style>