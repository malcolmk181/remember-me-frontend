<script>
    export let thing;

    let easyMDE;

    import { afterUpdate, onMount } from "svelte";
    import ThingThumbnail from "./ThingThumbnail.svelte";

    const toggleFavorite = async () => {
        fetch(`https://remember-me-rails.herokuapp.com/things/${thing.id}`, {
            method: 'PATCH',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                is_favorite: !thing.is_favorite
            })
        })
            .then(response => response.json())
            .then(data => {
                thing.is_favorite = data.is_favorite;
                thing.last_updated = data.last_updated;
            })
            .catch(console.log);
    };

    const saveContent = async () => {
        let newValue = easyMDE.value();
        if (newValue !== thing.content) {
            fetch(`https://remember-me-rails.herokuapp.com/things/${thing.id}`, {
                method: 'PATCH',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    content: newValue
                })
            })
                .then(response => response.json())
                .then(data => {
                    thing.content = data.content;
                    thing.last_updated = data.last_updated;
                })
                .catch(console.log);
        }
    };

    onMount(() => {
        easyMDE = new EasyMDE();
        easyMDE.value(thing.content);
    });

    afterUpdate(() => {
        easyMDE.value(thing.content);
    });

</script>

<div class='content' data-id={thing.id}>
    <h1>{thing.name}</h1>
    <textarea id="thing-text-area"></textarea>
    <button class='button' on:click|preventDefault="{saveContent}">Save</button>
    {#if thing.image_url}
        <img src="{thing.image_url}" alt="{thing.name} image"/>
    {/if}
    {#if thing.url}
        <a href="{thing.url}" target="_blank">{thing.url}</a>
    {/if}
    <p>Last updated at {thing.updated_at}</p>
    <button data-favorite="{thing.is_favorite}" class='button' on:click="{toggleFavorite}">{!!thing.is_favorite ? 'Favorited' : 'Not favorited'}</button>

    {#if thing.child_things.length > 0}
        <h1 class='title is-4'>Children:</h1>
        <div class='is-flex is-flex-direction-row is-flex-wrap-wrap'>
            {#each thing.child_things as child}
                <ThingThumbnail
                    name={child.name}
                    id={child.id}
                    on:message
                />
            {/each}
        </div>
    {/if}
</div>