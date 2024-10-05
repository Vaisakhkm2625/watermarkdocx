<script>
	import { Canvas, Rect, Circle, Text, StaticCanvas, Image as FabricImage } from 'fabric';
	import { onMount } from 'svelte';

	let mainFabricCanvas;
	let canvasIds = ['canvas1', 'canvas2']; // Add more IDs as needed
	let canvases = [];
	let inputText = '';
	let selectedImageFile = null; // For image uploads

	onMount(() => {
		mainFabricCanvas = new Canvas('mainFabricCanvas');

		addrect();

		canvasIds.forEach((id) => {
			canvases.push(new Canvas(id));
		});

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

	// Handle image uploads and set as background
	function setBackgroundImage() {
		if (selectedImageFile) {
			const reader = new FileReader();
			reader.onload = function (event) {
				const imgElement = new Image();
				imgElement.src = event.target.result;
				imgElement.onload = function () {
					const fabricImg = new FabricImage(imgElement);

					// Set the background image for the main canvas
					mainFabricCanvas.set({
						backgroundImage: fabricImg
					});
					mainFabricCanvas.requestRenderAll(); // Re-render the canvas

					// Set the background image for each canvas in the array
					canvases.forEach((canvas) => {
						canvas.set({
							backgroundImage: fabricImg
						});
						canvas.requestRenderAll(); // Re-render each canvas
					});
				};
			};
			reader.readAsDataURL(selectedImageFile);
		}
	}

	function applyChanges() {
		let objects = JSON.stringify(mainFabricCanvas.toJSON());
		console.log(objects);

		canvases.forEach((canvas) => {
			if (canvas !== mainFabricCanvas) {
				canvas.loadFromJSON(objects);
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
	<button on:click={addrect}>Add Rectangle</button>
	<button on:click={addcircle}>Add Circle</button>
	<input bind:value={inputText} />
	<button on:click={addText}>Add Text</button>

	<!-- Image upload section for background -->
	<input type="file" accept="image/*" on:change={(e) => (selectedImageFile = e.target.files[0])} />
	<button on:click={setBackgroundImage}>Set Background Image</button>

	<button on:click={applyChanges}>Apply Changes</button>
</div>
