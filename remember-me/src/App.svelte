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
						<ThingDetails bind:thing={thing}/>
						{#if thing.child_things.length > 0}
							<h1 class='title is-4'>Children:</h1>
							<div class='is-flex is-flex-direction-row is-flex-wrap-wrap'>
								{#each thing.child_things as child}
									<ThingThumbnail
										name={child.name}
										id={child.id}
										on:message="{() => changeThing(child)}"
									/>
								{/each}
							</div>
						{/if}
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