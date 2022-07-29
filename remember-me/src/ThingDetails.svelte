<script>
    export let thing;
    export let things;

    let easyMDE;
    let nameState = 'viewing';

    import { afterUpdate, onMount } from "svelte";
    import ThingThumbnail from "./ThingThumbnail.svelte";
    import ChildCreate from "./ChildCreate.svelte";

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

    const saveName = async (newName) => {
        if (thing.id) {
            fetch(`https://remember-me-rails.herokuapp.com/things/${thing.id}`, {
                method: 'PATCH',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    name: newName
                })
            })
                .then(response => response.json())
                .then(data => {
                    thing.name = data.name;
                    thing.last_updated = data.last_updated;
                })
                .catch(console.log);
        }
    };

    const saveContent = async () => {
        let newValue = easyMDE.value();
        if (thing.id) {
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
        } else {
            fetch('https://remember-me-rails.herokuapp.com/things', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    name: document.querySelector('#thing-name').innerHTML,
                    content: newValue
                })
            })
                .then(response => response.json())
                .then(data => thing = data)
                .catch(console.log);
        }
        
    };

    const toggleName = () => {
        const thingName = document.querySelector('#thing-name');
        if (nameState === 'viewing') {
            nameState = 'editing';
            thingName.contentEditable = true;
            thingName.focus();
        } else {
            nameState = 'viewing';
            thingName.contentEditable = false;
            if (thingName.innerHTML !== thing.name) {
                saveName(thingName.innerHTML);
            }
        }
    };

    const deleteThing = async () => {
        fetch(`https://remember-me-rails.herokuapp.com/things/${thing.id}`, {
            method: 'DELETE'
        })
            .then( () => thing = null )
            .catch(console.log);
    };

    const toggleCreateChild = () => {
        const childCreate = document.querySelector('#child-create');
        if (childCreate.classList.contains('is-hidden')) {
            childCreate.classList.remove('is-hidden');
        } else {
            childCreate.classList.add('is-hidden');
        }
    };

    const handleAddedChild = (event) => {
        toggleCreateChild();
        thing.child_things.push(things.find(thing => thing.id === event.detail.id));
        thing = thing;
    };

    onMount(() => {
        easyMDE = new EasyMDE({ previewImagesInEditor: true, spellChecker: false });
        easyMDE.value(thing.content);
        easyMDE.togglePreview();
    });

    afterUpdate(() => {
        easyMDE.value(thing.content);

        if (!easyMDE.isPreviewActive()) {
            easyMDE.togglePreview();
        }
    });

</script>

<div class='content' data-id={thing.id}>
    <nav class='level'>
        <div class='level-left'>
            <div class='level-item'>
                <h1 class='title' id="thing-name" data-name='{thing.name}' contenteditable="false">{thing.name}</h1>
            </div>
        </div>
        <div class='level-right'>
            <div class='level-item'>
                <button
                    class='button'
                    data-state='{nameState}'
                    id='edit-thing-name'
                    on:click|preventDefault="{toggleName}">
                        {nameState === 'viewing' ? 'Edit name' : 'Save name'}
                </button>
            </div>
        </div>
    </nav>
    <textarea id="thing-text-area"></textarea>
    
    {#if thing.image_url}
        <img src="{thing.image_url}" alt="{thing.name} image"/>
    {/if}
    {#if thing.url}
        <a href="{thing.url}" target="_blank">{thing.url}</a>
    {/if}
    {#if thing.updated_at}
    <p>Last updated at {thing.updated_at}</p>
    {/if}
    
    <nav class="level">
        <div class="level-left">
            {#if thing.id}
            <div class="level-item">
                <button 
                    data-favorite="{thing.is_favorite}"
                    class='button'
                    on:click="{toggleFavorite}">
                        {!!thing.is_favorite ? '❤️ Favorited' : 'Not favorited'}
                </button>
            </div>
            {/if}
            <div class="level-item">
                <button class='button' on:click|preventDefault="{saveContent}">Save content</button>
            </div>
            {#if thing.id}
                <div class="level-item">
                    <button class='button is-danger is-light' on:click|preventDefault="{deleteThing}">Delete this thing</button>
                </div>
            {/if}
        </div>
    </nav>

    {#if thing.id}
        <h1 class='title is-4'>Children:</h1>
        <div class='is-flex is-flex-direction-row is-flex-wrap-wrap'>
            {#each thing.child_things as child}
                <ThingThumbnail
                    name={child.name}
                    id={child.id}
                    on:message
                />
            {/each}
            <span class="tag is-medium is-clickable is-warning is-light" on:click="{toggleCreateChild}">New child</span>
        </div>

        <div id='child-create' class='is-hidden'>
            <ChildCreate
                things={things}
                thing={thing}
                on:done="{handleAddedChild}"
            />
        </div>
    {/if}
</div>

<style>
	.is-flex {
		gap: 1rem;
	}

    .is-hidden {
        display: none;
    }
</style>
