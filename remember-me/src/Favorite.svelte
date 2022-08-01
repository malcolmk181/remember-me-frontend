<script>
    export let favorite;
    export let thing;

    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function handleClick() {
		dispatch('message', {
			id: favorite.id
		});
	}
</script>

<!-- svelte-ignore a11y-missing-attribute -->
<a class="panel-block {(thing && thing.id === favorite.id) ? 'is-active' : ''}" data-id='{favorite.id}' on:click="{handleClick}">
    <span class="panel-icon">
        <i class="fa-solid fa-book" aria-hidden="true"></i>
    </span>
    {favorite.attributes.name}
</a>