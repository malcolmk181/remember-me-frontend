<script>
    export let name;
    export let id;
    export let content;
    export let image_url;
    export let url;
    export let is_favorite;
    export let updated_at;

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
    <button>{is_favorite ? 'Favorited' : 'Not favorited'}</button>

</div>