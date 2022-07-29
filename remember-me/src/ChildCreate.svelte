<script>
    export let things;
    export let thing;

    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

    let currentChildrenIds = thing.child_things.map(child => child.id);

    let selectableThings = things.filter(candidate => {
        return thing.id !== candidate.id && currentChildrenIds.includes(candidate.id) === false;
    })

    const saveChild = async () => {
        const childId = parseInt(document.querySelector('#child-select').value);

        fetch(`https://remember-me-rails.herokuapp.com/thing_relationships`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                child_thing_id: childId,
                parent_thing_id: thing.id
            })
        })
            .then(response => response.json())
            .then(data => {
                dispatch('done', {
                    id: childId
                });
            })
            .catch(console.log);
    };
</script>

<div class='level mt-3'>
    <div class='level-left'>
        <div class='level-item'>
            <p>Choose a thing</p>
        </div>
        <div class='level-item'>
            <div class="select">
                <select id='child-select'>
                    {#each selectableThings as localThing}
                        <option value={localThing.id}>{localThing.name}</option>
                    {/each}
                </select>
            </div>
        </div>
        <div class='level-item'>
            <button class='button' on:click|preventDefault="{saveChild}">Save</button>
        </div>
    </div>
</div>