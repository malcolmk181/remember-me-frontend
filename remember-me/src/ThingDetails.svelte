<script>
    export let thing;

    const toggleFavorite = () => {
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

</script>

<div class='content' data-id={thing.id}>
    <h1>{thing.name}</h1>
    <p>{thing.content}</p>
    {#if thing.image_url}
        <img src="{thing.image_url}" alt="{thing.name} image"/>
    {/if}
    {#if thing.url}
        <a href="{thing.url}" target="_blank">{thing.url}</a>
    {/if}
    <p>Last updated at {thing.updated_at}</p>
    <button data-favorite="{thing.is_favorite}" class='button' on:click="{toggleFavorite}">{!!thing.is_favorite ? 'Favorited' : 'Not favorited'}</button>
</div>