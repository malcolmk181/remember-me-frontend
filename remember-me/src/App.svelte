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

	// A stack to hold the ids of previously viewed things.
	let backStack = [];

	// when things is updated, set favorites
	const setFavorites = () => {
		favorites = things.filter(thing => thing.attributes.is_favorite);
	};

	// load the things from the backend server
	const getThings = async () => {
		return fetch('https://remember-me-rails.herokuapp.com/things')
			.then(response => response.json())
			.then(data => {
				things = data.data;
				setFavorites();
			});
	};

	// load a new thing into thing
	const changeThing = (newThing) => {
		// when the back button is clicked
		if (newThing === 'back') {
			// overwrite newThing as an object, with the id of the most recent thing in the back stack
			newThing = {
				id: backStack.pop(),
				goingBack: true
			};
			// force update of backStack
			backStack = backStack;
		}
		// valid thing id
		if (newThing && newThing.id) {
			// if loading a new thing (and not going backwards), add it to the back stack
			if (!newThing.goingBack && thing && thing.data.id) {
				backStack.push(thing.data.id);
				backStack = backStack;
			}

			// load in the new thing from the backend
			fetch(`https://remember-me-rails.herokuapp.com/things/${newThing.id}`)
				.then(response => response.json())
				.then(data => {
					thing = data;
				});
		}
		// if newThing is null, empty the backstack
		else {
			backStack = [];
			thing = newThing;
		}
	};

	// load an empty thing into ThingDetails by calling changeThing with a placeholder thing
	const showNewThing = () => {
		changeThing({
			data: {
				id: null,
				type: "things",
				attributes: {
					name: '',
					content: '',
					is_favorite: false,
					updated_at: null
				},
				relationships: {
					child_things: {
						data: []
					}
				}
			},
			edit: true
		});
	};

	// when thing is updated, reload the things list from the server
	$: getThings(thing);

	// get thing from the server on mount
	onMount(getThings);
</script>

<main>
	<!-- Navbar -->
	<section class='section'>
		<nav class="level">
			<!-- When the navbar is clicked, closes thingDetails by sending null to changeThing -->
			<div class="level-item has-text-centered is-clickable" on:click="{() => changeThing(null)}">
				<h1 class="title">Remember Me!</h1>
			</div>
		</nav>
	</section>
	
	<!-- Existing things & favorites section -->
	<section class='section'>
		<div class='container'>
			<div class='columns is-6'>
				<!-- Existing things & create thing button -->
				<div id='things' class='column'>
					<h1 class='title'>Things:</h1>
					<!-- Existing things -->
					<div class='is-flex is-flex-direction-row is-flex-wrap-wrap'>
						{#if things}
							{#each things as localThing}
								<ThingThumbnail
									name={localThing.attributes.name}
									id={localThing.id}
									on:message="{() => changeThing(localThing)}"
								/>
							{/each}
						{:else}
							<p>Loading things...</p>
						{/if}
					</div>
					<!-- Create thing button. Waits for things to load to show button -->
					<div class='is-flex is-flex-direction-row is-flex-wrap-wrap mt-6'>
						{#if things}
							<span class="tag is-primary is-light is-large is-clickable" on:click="{showNewThing}">Create new thing</span>
						{/if}
					</div>
				</div>
				<!-- Favorite things panel -->
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

	<!-- Thing details section -->
	<section class='section'>
		<div class='container'>
			<div id='show-thing'>
				<!-- Back button -->
				{#if backStack.length > 0}
					<div class='is-flex is-flex-direction-row mb-3'>
						<span class="tag is-info is-light is-medium is-clickable" on:click="{() => changeThing('back')}">← Back</span>
					</div>
				{/if}
				<!-- ThingDetails -->
				{#if thing}
					<div class='box'>
						<ThingDetails
							bind:thing={thing}
							things={things}
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