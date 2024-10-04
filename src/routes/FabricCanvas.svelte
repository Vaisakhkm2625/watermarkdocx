<script>
	import { onMount } from 'svelte';
	import { Canvas, Rect } from 'fabric'; // browsea

	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	let canvas;
	let fabricCanvas;

	export let id;
	export let syncObjects = () => {}; // Function to sync objects across canvases

	onMount(() => {
		fabricCanvas = new Canvas(id);
		dispatch('created');

		// Example: Add an object to canvas
		const rect = new Rect({
			left: 100,
			top: 100,
			fill: 'red',
			width: 50,
			height: 50
		});

		fabricCanvas.add(rect);

		// Listen for object changes (e.g., drag, scale, etc.)
		fabricCanvas.on('object:modified', (e) => {
			syncObjects(fabricCanvas);
		});

		fabricCanvas.on('object:moving', (e) => {
			syncObjects(fabricCanvas);
		});

		fabricCanvas.on('object:scaling', (e) => {
			syncObjects(fabricCanvas);
		});
	});

	export function updateObjects(objects) {
		fabricCanvas.clear();
		fabricCanvas.loadFromJSON(objects);
		fabricCanvas.requestRenderAll();
	}
</script>

<canvas bind:this={canvas} {id}></canvas>
