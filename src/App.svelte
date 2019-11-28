<script>
	const DB = 'https://www.jsonstore.io/8ee7336db06b263a98141280ccc41fadaf4b65db7b3a12d832631624c230a441';
	const imgurClientID = 'df14151b1226e69';
	
	import DBStatus from './DBStatus.svelte';
	import SaveNewView from './SaveNewView.svelte';
	import ImageList from './ImageListing.svelte';
	import { imagesStore } from './store.js';
	
	let isDBConnected = false;
	
	function onDBConnectionUpdate(e) {
		const ok = e.detail.ok;
		const payload = e.detail.payload;
		isDBConnected = ok;
	}
	
</script>

<DBStatus
	DB={DB}
  on:db_status_update={onDBConnectionUpdate}
/>

{#if isDBConnected}
	<SaveNewView DB={DB} skip_saving={false} clientID={imgurClientID} />
	<ImageList />
{/if}