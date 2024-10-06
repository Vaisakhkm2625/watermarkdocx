<script>
	import { Canvas, Rect, Circle, Text, Image as FabricImage } from 'fabric';
	import { onMount } from 'svelte';

	let mainFabricCanvas;
	// {id: canvas1, fabriccanvas: fabricCanvasInstance, sync: bool}
	let canvases = []; // No predefined canvas IDs, dynamically add them
	//let = [];
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

	const findCanvasById = (id) => {
		return canvases.find((canvas) => canvas.id === id);
	};

	const exportCanvas = (fabricCanvas) => {
		// Export the canvas to a data URL in PNG format with full resolution
		const dataURL = fabricCanvas.toDataURL({
			format: 'png',
			quality: 1, // Maximum quality for PNG
			multiplier: 1 // No scaling; export at full internal size
		});

		// Create a temporary link to trigger the download
		const link = document.createElement('a');
		link.href = dataURL;
		link.download = 'exported-image.png';
		document.body.appendChild(link);
		link.click();
		document.body.removeChild(link);
	};

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
					canvases = [...canvases, { id: newCanvasId, sync: true }]; // Update canvases array

					// Create a new canvas instance and set its background image
					setTimeout(() => {
						// Wait for canvas to be added to DOM
						const canvasBGImage = new FabricImage(imgElement);
						const newCanvas = new Canvas(newCanvasId, {
							width: imgElement.width,
							height: imgElement.height
						});
						//const canvasBGImage = new FabricImage(imgElement, {
						//	scaleX: newCanvas.width / imgElement.width,
						//	scaleY: newCanvas.height / imgElement.height
						//});

						newCanvas.set({ backgroundImage: canvasBGImage });
						//newCanvas.setDimensions({ width: imgElement.width, height: imgElement.height });
						newCanvas.requestRenderAll();
						//newCanvas.lowerCanvasEl.style.width = '400px';
						//newCanvas.lowerCanvasEl.style.height = '400px';

						newCanvas.canvasBGImage = canvasBGImage;
						findCanvasById(newCanvasId).fabriccanvas = newCanvas;
					}, 0);
				};
			};
			reader.readAsDataURL(selectedImageFile);
		}
	}

	function setBgAll() {
		canvases.forEach((canvas) => {
			console.log('canvas: ', { canvas });
			canvas.set({ backgroundImage: canvas.fabriccanvas.canvasBGImage }); // this is setting background image when i click button
			canvas.requestRenderAll();
		});
	}

	function syncCanvasObjects() {
		const objectsJSON = mainFabricCanvas.toJSON();
		console.log(objectsJSON);

		canvases.forEach((canvasItem) => {
			let canvas = canvasItem.fabriccanvas;

			if (canvas !== mainFabricCanvas && canvasItem.sync) {
				canvas.loadFromJSON(objectsJSON, (objects) => {
					canvas.requestRenderAll();
				});

				const scaleX = canvas.width / mainFabricCanvas.width;
				const scaleY = canvas.height / mainFabricCanvas.height;

				const scaleRatio = Math.min(canvas.height / mainFabricCanvas.height);

				setTimeout(() => {
					canvases;

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

				// canvasIds.forEach((c) => {
				//   canvas = c.fabriccanvas;
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

<div class="section">
	<div class="title">docx image batch editor</div>
	<div class="subtitle">batch Apply watermarks, add text etc in image in a docx file</div>
</div>
<div class="section">
	<input class="file-input" type="file" bind:this={fileInput} name="resume" accept=".docx" />
	<button onclick="processDocx()">Add Watermark</button>
	<a id="downloadLink" style="display: none;">Download Watermarked Docx</a>
</div>

<div class="section">
	<div class="container">
		<canvas id="mainFabricCanvas" class="canvas" width="500" height="300"></canvas>
		<div>
			<button on:click={addrect} class="button">Add Rectangle</button>
			<button on:click={addcircle} class="button">Add Circle</button>
			<input bind:value={inputText} class="input" />
			<button on:click={addText} class="button">Add Text</button>
			<input type="color" class="input" />
		</div>

		<!-- <button on:click={setBgAll} >setbg</button> -->

		<!-- Image upload section for background -->
		<input
			type="file"
			accept="image/*"
			on:change={(e) => (selectedImageFile = e.target.files[0])}
		/>
		<button on:click={setBackgroundImage}>Create Canvas with Background Image</button>

		<!--<button on:click={syncCanvasObjects}>Sync Canvas Objects</button>-->

		{#each canvases as canvas}
			<div>
				<br />
				<br />
				sync: {canvas.sync}
				<input type="checkbox" bind:checked={canvas.sync} />

				<button
					on:click={() => {
						exportCanvas(canvas.fabriccanvas);
					}}
					class="button"
				>
					Export
				</button>

				<canvas class="canvas" id={canvas.id}></canvas>
			</div>
		{/each}
	</div>
</div>

<style>
	.canvas {
		border: 1px solid white;
		width: 400px;
		height: 300px;
	}
</style>
