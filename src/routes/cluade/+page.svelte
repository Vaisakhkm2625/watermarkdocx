<script>
	import { onMount } from 'svelte';
	import { Canvas, Rect, Circle, Object } from 'fabric';
	import FabricCanvas from '../FabricCanvas.svelte';

	let canvas1, canvas2;

	onMount(() => {
		// Initialize the first canvas
		const fabricCanvas1 = new Canvas('canvas1');
		const fabricCanvas2 = new Canvas('canvas2');

		// Add some objects to the first canvas
		const rect = new Rect({
			left: 100,
			top: 100,
			fill: 'red',
			width: 50,
			height: 50
		});

		const circle = new Circle({
			left: 200,
			top: 100,
			fill: 'blue',
			radius: 25
		});

		fabricCanvas1.add(rect);
		fabricCanvas1.add(circle);

		// Function to transfer objects to the new canvas
		function transferObjects() {
			const objects = fabricCanvas1.getObjects();
			const json = fabricCanvas1.toObject(); // Get the objects in JSON format
			console.log(json);

			// Clear the second canvas
			fabricCanvas2.clear();

			// Load the objects into the second canvas
			//fabricCanvas2.loadFromJSON(json, fabricCanvas2.renderAll.bind(fabricCanvas2));

			Object.fromObject(json, function (newob) {
				console.log(newob);
				newob.forEach(function (obj) {
					console.log(obj);
					fabricCanvas2.add(obj);
				});
				fabricCanvas2.requestRenderAll();
			});
		}

		// Attach the transfer function to a button click
		document.getElementById('transferButton').addEventListener('click', transferObjects);
	});
</script>

<div>
	<h2>Canvas 1</h2>
	<canvas id="canvas1" width="400" height="400"></canvas>

	<button id="transferButton">Transfer Objects to Canvas 2</button>

	<h2>Canvas 2</h2>
	<canvas id="canvas2" width="400" height="400"></canvas>
</div>

<style>
	canvas {
		border: 1px solid black;
		margin: 10px;
	}
</style>
