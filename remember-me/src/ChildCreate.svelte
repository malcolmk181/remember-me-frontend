<script>
    export let things;
    export let thing;

    let currentChildrenIds, selectableThings;

    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

    // when thing is updated, update the currentChildrenIds
    $: currentChildrenIds = thing.included ? thing.included.map(child => child.id) : [];

    // when things is updated, update the selectableThings
    $: selectableThings = things.filter(candidate => {
        // only allow things that are not the current thing and not already a child of the current thing
        return thing.data.id !== candidate.id && currentChildrenIds.includes(candidate.id) === false;
    });

    // Creates a parent-child relationship between the current 'thing'
    // and the child selected in the 'selectableThings' list.
    const saveChild = async () => {
        const childId = parseInt(document.querySelector('#child-select').value);

        fetch(`https://remember-me-rails.herokuapp.com/thing_relationships`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                child_thing_id: childId,
                parent_thing_id: thing.data.id
            })
        })
            .then(response => response.json())
            .then(() => {
                // dispatch the done message to ThingDetails to update the UI
                dispatch('done', {
                    id: childId
                });
            })
            .catch(console.log);
    };
</script>

<!-- The Bulma "level" places several items in a horizontal element. -->
<!-- This level contains a select box for the possible children for this thing -->
<div class='level mt-3'>
    <div class='level-left'>
        <div class='level-item'>
            <p>Choose a thing</p>
        </div>
        <div class='level-item'>
            <div class="select">
                <select id='child-select'>
                    {#each selectableThings as localThing}
                        <option value={localThing.id}>{localThing.attributes.name}</option>
                    {/each}
                </select>
            </div>
        </div>
        <div class='level-item'>
            <button class='button' on:click|preventDefault="{saveChild}">Save</button>
        </div>
    </div>
</div>