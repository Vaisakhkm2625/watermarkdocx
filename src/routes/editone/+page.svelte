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

		mainFabricCanvas.on('object:modified', syncCanvasObjects);
		mainFabricCanvas.on('object:moving', syncCanvasObjects);
		mainFabricCanvas.on('object:scaling', syncCanvasObjects);
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
						//const canvasBGImage = new FabricImage(imgElement, {
						//	scaleX: newCanvas.width / imgElement.width,
						//	scaleY: newCanvas.height / imgElement.height
						//});
						const canvasBGImage = new FabricImage(imgElement);

						newCanvas.set({ backgroundImage: canvasBGImage });
						newCanvas.setDimensions({ width: imgElement.width, height: imgElement.height });
						newCanvas.requestRenderAll();

						newCanvas.canvasBGImage = canvasBGImage;
						canvases.push(newCanvas); // Push new canvas to canvases array
					}, 0);
				};
			};
			reader.readAsDataURL(selectedImageFile);
		}
	}

	function setbgall() {
		canvases.forEach((canvas) => {
			console.log('canvas: ', { canvas });
			canvas.set({ backgroundImage: canvas.canvasBGImage }); // this is setting background image when i click button
			canvas.requestRenderAll();
		});
	}

	// Synchronize objects (without background image) across canvases
	function syncCanvasObjects() {
		const objectsJSON = mainFabricCanvas.toJSON();
		console.log(objectsJSON);

		canvases.forEach((canvas) => {
			if (canvas !== mainFabricCanvas) {
				canvas.loadFromJSON(objectsJSON, (objects) => {
					canvas.requestRenderAll();
				});

				const scaleX = canvas.width / mainFabricCanvas.width;
				const scaleY = canvas.height / mainFabricCanvas.height;

				const scaleRatio = Math.min(canvas.height / mainFabricCanvas.height);

				setTimeout(() => {
					canvas.set({ backgroundImage: canvas.canvasBGImage });

					canvas.getObjects().forEach((obj) => {
						// Scale object's position and dimensions

						const scalePosX =
							(canvas.width - obj.width * obj.scaleX * scaleRatio) /
							(mainFabricCanvas.width - obj.width * obj.scaleX);

						const scalePosY =
							(canvas.height - obj.height * obj.scaleY * scaleRatio) /
							(mainFabricCanvas.height - obj.height * obj.scaleY);

						obj.set({
							left: obj.left * scalePosX,
							top: obj.top * scalePosY,
							scaleX: obj.scaleX * scaleRatio,
							scaleY: obj.scaleY * scaleRatio
						});

						obj.setCoords(); // Update the object's coordinates
					});

					canvas.requestRenderAll();
				}, 0);

				// canvases.forEach((canvas) => {
				// 	if (canvas !== mainFabricCanvas) {
				// 		console.log('before: ', { canvas });
				// 		canvas.loadFromJSON(objectsJSON, () => {
				// 			console.log('after: ', { canvas });
				// 			canvas.set({ backgroundImage: canvas.canvasBGImage }); // this is not setting bg image
				// 			canvas.requestRenderAll();
				// 		});
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
	<button on:click={setbgall}>setbg</button>

	<!-- Image upload section for background -->
	<input type="file" accept="image/*" on:change={(e) => (selectedImageFile = e.target.files[0])} />
	<button on:click={setBackgroundImage}>Create Canvas with Background Image</button>

	<button on:click={syncCanvasObjects}>Sync Canvas Objects</button>
</div>

<style>
	canvas {
		border: 1px solid black;
	}
</style>
