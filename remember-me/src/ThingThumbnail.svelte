<script>
    export let name;
    export let id;

    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function handleClick() {
		dispatch('message', {
			id: id,
            name: name
		});
	}
</script>

<span class="tag is-medium" data-id={id} on:click="{handleClick}">{name}</span>