<script>
	export let DB;
	
	import { imagesStore } from './store.js';
	
	let images = [];
	
	const unsubscribe = imagesStore.subscribe(value => images = value);
	
	let showError = false;
	
	async function fetchImages() {
		try {
			if(DB) {
				const res = await fetch(DB);
				if(res.ok) {
					const payload = await res.json();
					images = payload.result;
				}
			}
		} catch(err) {
			showError = true;
			console.error('Fail to fetch list of images');
			console.error(err);
		}
	}
	//fetchImages();
</script>

<style>
	.error {
		color: #FFF;
		background: #ef4949;
		padding: 0.4em;
	}
</style>

<div>
	<hr />
	
	{#if showError}
		<p class="error">
			Fail to get list of images
		</p>
	{/if}
	
	{#if images.length >= 1}
		{#each images as image}
			<img 
			  src={image.image}
				alt="User upload content"
			/>
		{/each}
	{:else}
		<p>
			No images
		</p>
	{/if}
</div>