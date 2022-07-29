<script>
	// build with 'npm run build'
	// then deploy with 'surge public'

	import { onMount } from "svelte"
	import ThingThumbnail from "./ThingThumbnail.svelte";
	import ThingDetails from "./ThingDetails.svelte";
	import Favorite from "./Favorite.svelte";

	let things;
	let thing;
	let favorites;

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

	const changeThing = (newThing) => {
		thing = newThing;
	};

	const showNewThing = () => {
		changeThing(thing = {
			id: null,
			name: '',
			content: '',
			is_favorite: false,
			last_updated: null,
			child_things: []
		});
	};

	$: getThings(thing);

	onMount(getThings);
</script>

<main>
	<section class='section'>
		<nav class="level">
			<div class="level-item has-text-centered is-clickable" on:click="{() => changeThing(null)}">
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
									on:message="{() => changeThing(localThing)}"
								/>
							{/each}
						{:else}
							<p>Loading things...</p>
						{/if}
					</div>
					<div class='is-flex is-flex-direction-row is-flex-wrap-wrap mt-3'>
						{#if things}
							<span class="tag is-primary is-light is-medium is-clickable" on:click="{showNewThing}">Create new thing</span>
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
								<Favorite
									thing={thing}
									favorite={favorite}
									on:message="{() => changeThing(favorite)}"
								/>
							{/each}
						{:else}
							<p class="panel-block"><em> No favorites yet. </em></p>
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
						<ThingDetails
							bind:thing={thing}
							on:message={(message) => changeThing(things.find(thing => thing.id === message.detail.id))}
						/>
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