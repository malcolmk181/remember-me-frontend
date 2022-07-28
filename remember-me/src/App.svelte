<script>
	// build with 'npm run build'
	// then cd into public, then do 'surge'

	import { onMount } from "svelte"
	import ThingThumbnail from "./ThingThumbnail.svelte";
	import ThingDetails from "./ThingDetails.svelte";

	let things;
	let thing;
	let favorites;

	const showThing = document.querySelector("#show-thing");

	const setFavorites = () => {
		favorites = things.filter(thing => thing.is_favorite);
	};

	const getThings = async () => {
		return fetch('https://remember-me-rails.herokuapp.com/things')
			.then(response => response.json())
			.then(data => {
				things = data;
				setFavorites();
			});
	};

	$: getThings(thing);

	onMount(getThings);
</script>

<main>
	<section class='section'>
		<nav class="level">
			<div class="level-item has-text-centered is-clickable" on:click="{() => thing = null}">
				<h1 class="title">Remember Me!</h1>
			</div>
		</nav>
	</section>
	
	<section class='section'>
		<div class='container'>
			<div class='columns is-6'>
				<div id='things' class='column'>
					<h1 class='title'>Things:</h1>
					<div class='is-flex is-flex-direction-row is-flex-wrap-wrap'>
						{#if things}
							{#each things as localThing}
								<ThingThumbnail
									name={localThing.name}
									id={localThing.id}
									on:message="{() => thing = localThing}"
								/>
							{/each}
						{/if}
					</div>
				</div>
				<div id='favorites' class='column is-one-quarter'>
					<article class="panel is-warning">
						<p class="panel-heading">
						  Favorites
						</p>
						{#if favorites}
							{#each favorites as favorite}
								<!-- svelte-ignore a11y-missing-attribute -->
								<a class="panel-block {(thing && thing.id === favorite.id) ? 'is-active' : ''}" data-id='{favorite.id}'
									on:click="{() => thing = favorite}">
									<span class="panel-icon">
										<i class="fas fa-book" aria-hidden="true"></i>
									</span>
									{favorite.name}
								</a>
							{/each}
						{/if}
					  </article>
				</div>
			</div>
		</div>
	</section>

	<section class='section'>
		<div class='container'>
			<div id='show-thing'>
				{#if thing}
					<div class='box'>
						<ThingDetails bind:thing={thing} />
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