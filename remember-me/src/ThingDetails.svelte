<script>
    export let name;
    export let id;
    export let content;
    export let image_url;
    export let url;
    export let is_favorite;
    export let updated_at;

    const toggleFavorite = () => {
        fetch(`https://remember-me-rails.herokuapp.com/things/${id}`, {
            method: 'PATCH',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                is_favorite: !is_favorite
            })
        })
            .then(response => response.json())
            .then(data => {
                is_favorite = data.is_favorite;
            })
    };

</script>

<div class='content' data-id={id}>
    <h1>{name}</h1>
    <p>{content}</p>
    {#if image_url}
        <img src="{image_url}" alt="{name} image"/>
    {/if}
    {#if url}
        <a href="{url}" target="_blank">{url}</a>
    {/if}
    <p>Last updated at {updated_at}</p>
    <button data-favorite="{is_favorite}" class='button' on:click="{toggleFavorite}">{!!is_favorite ? 'Favorited' : 'Not favorited'}</button>
</div>