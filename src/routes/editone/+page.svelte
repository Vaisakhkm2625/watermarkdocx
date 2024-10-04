<script>
	import { Canvas, Rect, Circle, Text, StaticCanvas } from 'fabric'; // browsea
	import { onMount } from 'svelte';

	let mainFabricCanvas;
	let canvasIds = ['canvas1', 'canvas2']; // Add more IDs as needed
	let canvases = [];

	onMount(() => {
		mainFabricCanvas = new Canvas('mainFabricCanvas');

		addrect();

		canvasIds.forEach((id) => {
			//canvases.push(new StaticCanvas(id));
			canvases.push(new Canvas(id));
		});

		mainFabricCanvas.on('object:modified', (e) => {
			applyChages();
		});

		mainFabricCanvas.on('object:moving', (e) => {
			applyChages();
		});

		mainFabricCanvas.on('object:scaling', (e) => {
			applyChages();
		});

		// Example: Add an object to canvas
	});
	function addrect() {
		const rect = new Rect({
			left: 100,
			top: 100,
			fill: 'red',
			width: 50,
			height: 50
		});

		mainFabricCanvas.add(rect);
	}

	function addcircle() {
		const circle = new Circle({
			radius: 20,
			fill: 'green',
			left: 100,
			top: 100
		});

		mainFabricCanvas.add(circle);
	}

	let inputText = '';

	function addText() {
		const text = new Text(inputText, {
			left: 100,
			top: 100,
			fill: 'black'
		});

		mainFabricCanvas.add(text);
	}

	function applyChages() {
		let objects = JSON.stringify(mainFabricCanvas.toJSON());
		console.log(objects);

		canvases.forEach((canvas) => {
			if (canvas !== mainFabricCanvas) {
				canvas.loadFromJSON(objects);
				//canvas.renderAll();
				canvas.requestRenderAll();
			}
		});
	}
</script>

<div>
	<canvas id="mainFabricCanvas"></canvas>
	{#each canvasIds as id}
		<canvas {id}></canvas>
	{/each}
	<button on:click={addrect}>add rectangle </button>
	<button on:click={addcircle}>add circle</button>
	<input bind:value={inputText} />
	{inputText}
	<button on:click={addText}>add Text</button>
	<button on:click={applyChages}>Apply changes</button>
</div>
