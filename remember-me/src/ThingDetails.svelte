<script>
    export let thing;
    export let things;

    let easyMDE;
    let nameState = 'viewing';

    import { afterUpdate, onMount } from "svelte";
    import ThingThumbnail from "./ThingThumbnail.svelte";
    import ChildCreate from "./ChildCreate.svelte";

    const toggleFavorite = async () => {
        fetch(`https://remember-me-rails.herokuapp.com/things/${thing.data.id}`, {
            method: 'PATCH',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                is_favorite: !thing.data.attributes.is_favorite
            })
        })
            .then(response => response.json())
            .then(data => {
                thing.data.attributes.is_favorite = data.data.attributes.is_favorite;
                thing.data.attributes.updated_at = data.data.attributes.last_updated;
            })
            .catch(console.log);
    };

    const saveName = async (newName) => {
        if (thing.data.id) {
            fetch(`https://remember-me-rails.herokuapp.com/things/${thing.data.id}`, {
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
                    thing.data.attributes.name = data.data.attributes.name;
                    thing.data.attributes.updated_at = data.data.attributes.updated_at;
                })
                .catch(console.log);
        }
    };

    const saveContent = async () => {
        let newValue = easyMDE.value();
        if (thing.data.id) {
            if (newValue !== thing.data.attributes.content) {
                fetch(`https://remember-me-rails.herokuapp.com/things/${thing.data.id}`, {
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
                        thing.data.attributes.content = data.data.attributes.content;
                        thing.data.attributes.updated_at = data.data.attributes.updated_at;
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
            if (thingName.innerHTML !== thing.data.attributes.name) {
                saveName(thingName.innerHTML);
            }
        }
    };

    const deleteThing = async () => {
        fetch(`https://remember-me-rails.herokuapp.com/things/${thing.data.id}`, {
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

        if (!thing.included) {
            thing.included = [];
        }

        thing.included.push(things.find(thing => thing.id === event.detail.id));
        thing = thing;
    };

    const deleteChild = async (message) => {
        const childId = parseInt(message.detail.id);
        
        fetch(`https://remember-me-rails.herokuapp.com/thing_relationships`)
            .then( (resp) => resp.json() )
            .then( (data) => {

                const thingRelationships = data.data;
                const relationship = thingRelationships.find(itm => parseInt(itm.attributes.child_thing_id) === childId && parseInt(itm.attributes.parent_thing_id) === parseInt(thing.data.id));


                if (relationship) {
                    fetch(`https://remember-me-rails.herokuapp.com/thing_relationships/${relationship.id}`, {
                        method: 'DELETE'
                    })
                        .then( () => {
                            thing.included = thing.included.filter(itm => parseInt(itm.id) !== childId);
                            thing = thing;
                        } )
                        .catch(console.log);
                }
            } )
            .catch(console.log);
    };

    onMount(() => {
        easyMDE = new EasyMDE({ previewImagesInEditor: true, spellChecker: false });
        easyMDE.value(thing.data.attributes.content);
        easyMDE.togglePreview();
    });

    afterUpdate(() => {
        easyMDE.value(thing.data.attributes.content);

        if (!easyMDE.isPreviewActive()) {
            easyMDE.togglePreview();
        }

        document.querySelector('#thing-name').innerHTML = thing.data.attributes.name ? thing.data.attributes.name : "";
    });

</script>

<div class='content' data-id={thing.data.id}>
    <nav class='level'>
        <div class='level-left'>
            <div class='level-item'>
                <h1 class='title' id="thing-name" data-name='{thing.data.attributes.name}' contenteditable="false">{thing.data.attributes.name}</h1>
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
    {#if thing.data.attributes.updated_at}
        <p>Last updated at {thing.data.attributes.updated_at}</p>
    {/if}
    
    <nav class="level">
        <div class="level-left">
            {#if thing.data.id}
                <div class="level-item">
                    <button 
                        data-favorite="{thing.data.attributes.is_favorite}"
                        class='button'
                        on:click="{toggleFavorite}">
                            {!!thing.data.attributes.is_favorite ? '❤️ Favorited' : 'Not favorited'}
                    </button>
                </div>
            {/if}
            <div class="level-item">
                <button class='button' on:click|preventDefault="{saveContent}">Save content</button>
            </div>
            {#if thing.data.id}
                <div class="level-item">
                    <button class='button is-danger is-light' on:click|preventDefault="{deleteThing}">Delete this thing</button>
                </div>
            {/if}
        </div>
    </nav>

    {#if thing.data.id}
        <h1 class='title is-4'>Children:</h1>
        <div class='is-flex is-flex-direction-row is-flex-wrap-wrap'>
            {#if thing.included}
                {#each thing.included as child}
                    <ThingThumbnail
                        name={child.attributes.name}
                        id={child.id}
                        deletable={true}
                        on:message
                        on:delete={deleteChild}
                    />
                {/each}
            {/if}
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
