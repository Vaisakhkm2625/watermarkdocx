<script>
	import FabricCanvas from './FabricCanvas.svelte';
	let canvases = [];
	let objects = {};

	// Function to collect references of FabricCanvas instances
	function addCanvas(canvas) {
		console.log('adding canvas');
		canvases.push(canvas);
	}

	function syncObjects(sourceCanvas) {
		// Extract all objects from the source canvas
		objects = JSON.stringify(sourceCanvas.toJSON());
		console.log(sourceCanvas.toJSON());

		// Sync the state with all other canvases
		console.log(canvases);
		canvases.forEach((canvas) => {
			if (canvas !== sourceCanvas) {
				canvas.updateObjects(objects);
			}
		});
	}
	let component;

	let canvasIds = ['canvas1', 'canvas2', 'canvas3']; // Add more IDs as needed
</script>

<div>
	{#each canvasIds as id}
		<FabricCanvas
			bind:this={component}
			on:created={() => addCanvas(component)}
			{id}
			{syncObjects}
		/>
		<!-- <FabricCanvas on:created={() => addCanvas(this)} {id}  onCreated={addCanvas} {syncObjects} /> -->
	{/each}
</div>
