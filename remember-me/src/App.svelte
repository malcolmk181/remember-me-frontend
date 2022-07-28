<script>
	// build with 'npm run build'
	// then cd into public, then do 'surge'

	import ThingThumbnail from "./ThingThumbnail.svelte";
	import ThingDetails from "./ThingDetails.svelte";

	let things;
	let thing;

	const showThing = document.querySelector("#show-thing");

	const getThings = async () => {
		return fetch('https://remember-me-rails.herokuapp.com/things')
			.then(response => response.json())
			.then(data => {
				things = data;
			});
	};

	const getThing = async (id) => {
		return fetch(`https://remember-me-rails.herokuapp.com/things/${id}`)
			.then(response => response.json())
	};

	const handleMessage = (event) => {
		const id = event.detail.id;
		// const name = event.detail.name;
		// alert(`${name} with id (${id}) was clicked!`);

		getThing(id).then(data => {
			thing = data;
		})
	};
</script>

<main>
	<section class='section'>
		<nav class="level">
			<div class="level-item has-text-centered" on:click="{() => thing = null}">
				<h1 class="title">Remember Me!</h1>
			</div>
		</nav>
	</section>
	
	<section class='section'>
		<div class='container'>
			<div id='things'>
				<h1 class='title'>Things:</h1>
				<div class='is-flex is-flex-direction-row is-flex-wrap-wrap is-justify-content-center'>
					{#await getThings()}
						<p>Loading your things...</p>
					{:then}
						{#each things as thing}
							<ThingThumbnail name={thing.name} id={thing.id} on:message="{handleMessage}"/>
						{/each}
					{:catch error}
						<p>Error loading things: {error}</p>
					{/await}
				</div>
			</div>
		</div>
	</section>

	<section class='section'>
		<div class='container'>
			<div id='show-thing'>
				{#if thing}
					<div class='box'>
						<ThingDetails name={thing.name} id={thing.id} content={thing.content} image_url={thing.image_url} url={thing.url} is_favorite={thing.is_favorite} updated_at={thing.updated_at}/>
					</div>
				{/if}
			</div>
		</div>
	</section>
</main>

<style>
	.is-flex {
		gap: 1rem;
	}
</style>