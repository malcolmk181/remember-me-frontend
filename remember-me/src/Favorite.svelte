<script>
    export let favorite;
    export let thing;

    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

    // Dispatch the id of a thing to App to load it into ThingDetails
	function handleClick() {
		dispatch('message', {
			id: favorite.id
		});
	}
</script>

<!-- An item in the favorites panel. Uses Bulma's "panel" element. -->
<!-- The 'is-active' class changes the icon's color to match the color of the panel. -->
<!-- svelte-ignore a11y-missing-attribute -->
<a class="panel-block {(thing && thing.data.id === favorite.id) ? 'is-active' : ''}" data-id='{favorite.id}' on:click="{handleClick}">
    <span class="panel-icon">
        <i class="fa-solid fa-book" aria-hidden="true"></i>
    </span>
    {favorite.attributes.name}
</a>