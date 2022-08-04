<script>
    export let name;
    export let id;
	export let deletable;

    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function handleClick() {
		dispatch('message', {
			id: id,
            name: name
		});
	}

	function handleDelete() {
		dispatch('delete', {
			id: id
		});
	}
</script>

<span class="tag is-medium is-clickable" data-id={id} on:click="{handleClick}">
	{name}
	{#if deletable}
		<span class="icon is-small ml-3">
			<i class="fas fa-times" on:click|stopPropagation="{handleDelete}"></i>
		</span>
	{/if}
</span>