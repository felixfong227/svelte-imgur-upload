<script>
    export let DB;

    import { createEventDispatcher } from 'svelte';
    import { imagesStore } from './store.js';

    let dbStatus = 'pending';
    let dbText = 'Connecting';

    const dispatcher = createEventDispatcher();

    async function checkDBStatus() {
        if (typeof DB !== 'string') throw new Error('Incorrect DB type');
        const res = await fetch(DB);
        if (res.ok === false) throw new Error(await res.text());
        return await res.json();
    }

    checkDBStatus()
        .then(payload => {
            imagesStore.update(images => (images = payload.result || []));
            dispatcher('db_status_update', {
                ok: true,
            });
            dbStatus = 'success';
            console.log(`DB connected ${DB}`);
        })
        .catch(err => {
            dispatcher('db_status_update', {
                ok: false,
            });
            dbStatus = 'error';
            dbText = 'Connection Fail: Database';
            console.error(err);
        });
</script>

<style lang="scss">
    @import url('https://fonts.googleapis.com/css?family=Alatsi&display=swap');

    .db-status {
        background: orange;
        color: #fff;
        font-family: 'Alatsi', sans-serif;
        padding: 0.5em;

        &.error {
            background: red;
        }

        &.success {
            background: lightgreen;
        }
    }
</style>

<div class="db-status-container">

    {#if dbStatus !== 'success'}
        <div class="db-status {dbStatus}">{dbText}</div>
    {/if}

</div>
