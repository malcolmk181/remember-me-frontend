<script>
    export let thing;
    export let things;

    let easyMDE;
    let nameState = 'viewing';

    import { afterUpdate, onMount } from "svelte";
    import ThingThumbnail from "./ThingThumbnail.svelte";
    import ChildCreate from "./ChildCreate.svelte";

    // Toggle the favorited status of thing
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
                // Update the attributes of thing from the backend response
                thing.data.attributes.is_favorite = data.data.attributes.is_favorite;
                thing.data.attributes.updated_at = data.data.attributes.last_updated;
            })
            .catch(console.log);
    };

    // Given a new name, update the name of thing in the backend
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
                    // Update thing with the backend response
                    thing.data.attributes.name = data.data.attributes.name;
                    thing.data.attributes.updated_at = data.data.attributes.updated_at;
                })
                .catch(console.log);
        }
    };

    // Save the content of the markdown editor as the content of thing in the backend
    const saveContent = async () => {
        let newValue = easyMDE.value();
        // if this is an existing thing, update it, otherwise create it
        if (thing.data.id) {
            // Check that content has actually changed before sending a request to the backend
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
                        // Update thing with the backend response
                        thing.data.attributes.content = data.data.attributes.content;
                        thing.data.attributes.updated_at = data.data.attributes.updated_at;
                    })
                    .catch(console.log);
            }
        } else {
            // create this thing & load it into the thing variable
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
                .then(data => thing = data) // update thing with the backend response
                .catch(console.log);
        }
        
    };

    // Toggle the edit mode of the name of thing
    const toggleName = () => {
        const thingName = document.querySelector('#thing-name');
        if (nameState === 'viewing') {
            // set name as editable & focus it
            nameState = 'editing';
            thingName.contentEditable = true;
            thingName.focus();
        } else {
            // set name as non-editable and save it to backend
            nameState = 'viewing';
            thingName.contentEditable = false;
            if (thingName.innerHTML !== thing.data.attributes.name) {
                saveName(thingName.innerHTML);
            }
        }
    };

    // delete the current thing in the backend and set thing as null
    const deleteThing = async () => {
        fetch(`https://remember-me-rails.herokuapp.com/things/${thing.data.id}`, {
            method: 'DELETE'
        })
            .then( () => thing = null )
            .catch(console.log);
    };

    // toggle the visibility of the createChild prompt
    const toggleCreateChild = () => {
        const childCreate = document.querySelector('#child-create');
        if (childCreate.classList.contains('is-hidden')) {
            childCreate.classList.remove('is-hidden');
        } else {
            childCreate.classList.add('is-hidden');
        }
    };

    // given the event from ChildCreate, update thing with the new child
    const handleAddedChild = (event) => {
        toggleCreateChild();

        if (!thing.included) {
            thing.included = [];
        }

        thing.included.push(things.find(thing => parseInt(thing.id) === parseInt(event.detail.id)));

        // force reload of thing
        thing = thing;
    };

    // given a delete message from ThingThumbnail, remove the child in backend and frontend
    const deleteChild = async (message) => {
        const childId = parseInt(message.detail.id);
        
        // first fetch all the thing_relationships
        fetch(`https://remember-me-rails.herokuapp.com/thing_relationships`)
            .then( (resp) => resp.json() )
            .then( (data) => {

                const thingRelationships = data.data;

                // fint the thing_relationship that matches the childId & thing
                const relationship = thingRelationships.find(itm => parseInt(itm.attributes.child_thing_id) === childId && parseInt(itm.attributes.parent_thing_id) === parseInt(thing.data.id));

                if (relationship) {
                    // delete the thing_relationship in the backend
                    fetch(`https://remember-me-rails.herokuapp.com/thing_relationships/${relationship.id}`, {
                        method: 'DELETE'
                    })
                        .then( () => {
                            // update thing with the backend response
                            thing.included = thing.included.filter(itm => parseInt(itm.id) !== childId);
                            thing = thing;
                        } )
                        .catch(console.log);
                }
            } )
            .catch(console.log);
    };

    // onMount, setup the markdown editor
    onMount(() => {
        easyMDE = new EasyMDE({ previewImagesInEditor: true, spellChecker: false });
        easyMDE.value(thing.data.attributes.content);
        easyMDE.togglePreview();
    });

    // after update of ThingDetails, update the content of markdown editor and #thing-name h1
    afterUpdate(() => {
        // update markdown editor with current thing content
        easyMDE.value(thing.data.attributes.content);

        // make sure markdown editor is in preview mode when loading a new thing
        if (!easyMDE.isPreviewActive()) {
            easyMDE.togglePreview();
        }

        // make sure the name h1 contains only the name of the current thing
        document.querySelector('#thing-name').innerHTML = thing.data.attributes.name ? thing.data.attributes.name : "";
    });

</script>

<div class='content' data-id={thing.data.id}>
    <!-- Bulma level that contains the name & edit-name button -->
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

    <!-- This textarea is actually the markdown editor. Variable easyMDE is taking control of this textarea -->
    <textarea id="thing-text-area"></textarea>

    {#if thing.data.attributes.updated_at}
        <p>Last updated at {thing.data.attributes.updated_at}</p>
    {/if}
    
    <!-- favorite, edit/save, and delete buttons -->
    <nav class="level">
        <div class="level-left">
            {#if thing.data.id}
            <!-- favorite button -->
                <div class="level-item">
                    <button 
                        data-favorite="{thing.data.attributes.is_favorite}"
                        class='button'
                        on:click="{toggleFavorite}">
                            {!!thing.data.attributes.is_favorite ? '❤️ Favorited' : 'Not favorited'}
                    </button>
                </div>
            {/if}
            <!-- save button -->
            <div class="level-item">
                <button class='button' on:click|preventDefault="{saveContent}">Save content</button>
            </div>
            {#if thing.data.id}
            <!-- delete button -->
                <div class="level-item">
                    <button class='button is-danger is-light' on:click|preventDefault="{deleteThing}">Delete this thing</button>
                </div>
            {/if}
        </div>
    </nav>

    <!-- list of children & create child button -->
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
            <!-- open the ChildCreate menu -->
            <span class="tag is-medium is-clickable is-warning is-light" on:click="{toggleCreateChild}">New child</span>
        </div>

        <!-- add new child menu - hidden by default -->
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
