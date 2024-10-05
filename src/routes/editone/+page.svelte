<script>
	import { onMount } from 'svelte';
	import { Canvas, Rect, Circle, FabricText, FabricImage } from 'fabric';

	let mainFabricCanvas;
	let canvasIds = ['canvas1', 'canvas2']; // Add more IDs as needed
	let canvases = [];
	let inputText = '';

	onMount(() => {
		// Initialize the main Fabric canvas
		mainFabricCanvas = new Canvas('mainFabricCanvas');
		addrect();

		// Initialize additional canvases
		canvasIds.forEach((id) => {
			canvases.push(new Canvas(id));
		});

		// Sync modifications across all canvases
		mainFabricCanvas.on('object:modified', applyChanges);
		mainFabricCanvas.on('object:moving', applyChanges);
		mainFabricCanvas.on('object:scaling', applyChanges);
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

	function addText() {
		const text = new Text(inputText, {
			left: 100,
			top: 100,
			fill: 'black'
		});
		mainFabricCanvas.add(text);
	}

	function applyChanges() {
		let objects = JSON.stringify(mainFabricCanvas.toJSON());

		// Sync the changes to other canvases
		canvases.forEach((canvas) => {
			if (canvas !== mainFabricCanvas) {
				canvas.loadFromJSON(objects, canvas.renderAll.bind(canvas));
			}
		});
	}

	// Function to handle image file uploads
	function handleImageUpload(event) {
		const files = event.target.files;

		// Load each image file and set as background for each canvas
		for (let i = 0; i < files.length; i++) {
			const reader = new FileReader();
			reader.onload = function (e) {
				const img = new window.Image();
				img.src = e.target.result;
				img.onload = function () {
					if (canvases[i]) {
						const fabricImg = new FabricImage(img, {
							left: 0,
							top: 0
						});
						canvases[i].setBackgroundImage(fabricImg, canvases[i].renderAll.bind(canvases[i]));
					}
				};
			};
			reader.readAsDataURL(files[i]);
		}
	}
</script>

<div>
	<canvas id="mainFabricCanvas" width="500" height="500"></canvas>
	{#each canvasIds as id}
		<canvas {id} width="500" height="500"></canvas>
	{/each}

	<!-- UI controls -->
	<button on:click={addrect}>Add Rectangle</button>
	<button on:click={addcircle}>Add Circle</button>
	<input bind:value={inputText} />
	{inputText}
	<button on:click={addText}>Add Text</button>
	<button on:click={applyChanges}>Apply Changes</button>

	<!-- File input for multiple images -->
	<input type="file" multiple on:change={handleImageUpload} />
</div>
