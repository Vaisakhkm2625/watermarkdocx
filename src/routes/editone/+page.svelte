<script>
	import { Canvas, Rect, Circle, Text, Image as FabricImage } from 'fabric';
	import { onMount } from 'svelte';

	let mainFabricCanvas;
	let canvasIds = []; // No predefined canvas IDs, dynamically add them
	let canvases = [];
	let inputText = '';
	let selectedImageFile = null; // For image uploads

	onMount(() => {
		mainFabricCanvas = new Canvas('mainFabricCanvas');

		addrect();

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

	// Create a new canvas for each uploaded image and set a unique background image
	function setBackgroundImage() {
		if (selectedImageFile) {
			const reader = new FileReader();
			reader.onload = function (event) {
				const imgElement = new Image();
				imgElement.src = event.target.result;
				imgElement.onload = function () {
					// Dynamically generate a new canvas ID
					const newCanvasId = `canvas${canvases.length + 1}`;
					canvasIds = [...canvasIds, newCanvasId]; // Update canvasIds array

					// Create a new canvas instance and set its background image
					setTimeout(() => {
						// Wait for canvas to be added to DOM
						const newCanvas = new Canvas(newCanvasId);
						newCanvas.set({
							backgroundImage: new FabricImage(imgElement, {
								scaleX: newCanvas.width / imgElement.width,
								scaleY: newCanvas.height / imgElement.height
							})
						});
						newCanvas.requestRenderAll();
						canvases.push(newCanvas); // Push new canvas to canvases array
					}, 0);
				};
			};
			reader.readAsDataURL(selectedImageFile);
		}
	}

	// This function replicates the main canvas content to other canvases
	function applyChanges() {
		let objects = JSON.stringify(mainFabricCanvas.toJSON());
		console.log(objects);

		canvases.forEach((canvas) => {
			// Only replicate objects, not background images
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
	<button on:click={setBackgroundImage}>Create Canvas with Background Image</button>

	<button on:click={applyChanges}>Apply Changes to Other Canvases</button>
</div>
