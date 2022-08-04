<script>
    export let name;
    export let id;
	export let deletable;

    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	// Dispatch the id of a thing to App to load it into ThingDetails
	function handleClick() {
		dispatch('message', {
			id: id,
            name: name
		});
	}

	// Dispatch the id of the child to ThingDetails for deletion of the relationship
	function handleDelete() {
		dispatch('delete', {
			id: id
		});
	}
</script>

<!-- Thing thumbnail. Uses Bulma's "tag" element -->
<span class="tag is-medium is-clickable" data-id={id} on:click="{handleClick}">
	{name}
	{#if deletable}
		<!-- Delete button -->
		<span class="icon is-small ml-3">
			<i class="fas fa-times" on:click|stopPropagation="{handleDelete}"></i>
		</span>
	{/if}
</span>