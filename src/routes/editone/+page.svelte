<script>
	import { Canvas, Rect, Circle, Text, Image as FabricImage } from 'fabric';
	import { onMount } from 'svelte';

	let mainFabricCanvas;
	// {id: canvas1, fabriccanvas: fabricCanvasInstance, sync: bool}
	let canvases = []; // No predefined canvas IDs, dynamically add them
	//let = [];
	let inputText = '';
	let selectedImageFile = null; // For image uploads
	let style = '-webkit-transform: scale(1);';

	onMount(() => {
		mainFabricCanvas = new Canvas('mainFabricCanvas');

		addrect();

		mainFabricCanvas.on('object:modified', syncCanvasObjects);
		mainFabricCanvas.on('object:moving', syncCanvasObjects);
		mainFabricCanvas.on('object:scaling', syncCanvasObjects);
	});
	let fileInput;

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

	function resizeCanvas(canvas, outerCanvasContainer) {
		//const outerCanvasContainer = outerCanvasContainer;

		const ratio = canvas.getWidth() / canvas.getHeight();
		const containerWidth = outerCanvasContainer.clientWidth;
		const containerHeight = outerCanvasContainer.clientHeight;

		const scale = containerWidth / canvas.getWidth();
		const zoom = canvas.getZoom() * scale;
		canvas.setDimensions({ width: containerWidth, height: containerWidth / ratio });
		canvas.setViewportTransform([zoom, 0, 0, zoom, 0, 0]);
	}

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

<section class="section">
	<div class="container">
		<h1 class="title">Docx Image Batch Editor</h1>
		<h2 class="subtitle">Batch apply watermarks, add text, etc. to images in a docx file.</h2>

		<!-- File Input -->
		<div class="field">
			<div class="file has-name is-fullwidth">
				<label class="file-label">
					<input
						class="file-input"
						type="file"
						bind:this={fileInput}
						name="resume"
						accept=".docx"
					/>
					<span class="file-cta">
						<span class="file-label">Choose a docx file…</span>
					</span>
					<span class="file-name">No file chosen</span>
				</label>
			</div>
		</div>

		<div class="field">
			<div class="control">
				<!-- <button class="button is-primary" on:click={processDocx}>Add Watermark</button> -->
				<button class="button is-primary">Add Watermark</button>
			</div>
		</div>

		<a id="downloadLink" style="display: none;">Download Watermarked Docx</a>
	</div>
</section>

<section class="section">
	<div class="container">
		<div class="columns">
			<!-- Left Column: Main Canvas -->
			<div class="column is-two-thirds">
				<!--<div class="box"> -->
				<canvas
					id="mainFabricCanvas"
					class="main-canvas"
					width={`${window.screen.width * (2 / 3) - 100}px`}
					height={`${window.screen.height * (2 / 3)}px`}
				></canvas>
				<!-- </div>-->
			</div>

			<!-- Right Column: Controls in a Card -->
			<div class="column is-one-third">
				<div class="card">
					<header class="card-header">
						<p class="card-header-title">Canvas Controls</p>
					</header>

					<div class="card-content">
						<!-- Add Shapes and Text -->
						<div class="buttons">
							<button class="button is-link is-fullwidth" on:click={addrect}>Add Rectangle</button>
							<button class="button is-link is-fullwidth" on:click={addcircle}>Add Circle</button>
						</div>

						<!-- Add Text -->
						<div class="field">
							<label class="label">Add Text</label>
							<div class="control">
								<input class="input" bind:value={inputText} placeholder="Type text" />
							</div>
							<div class="control">
								<button class="button is-link is-fullwidth" on:click={addText}>Add Text</button>
							</div>
						</div>

						<!-- Pick Color -->
						<div class="field">
							<label class="label">Pick Color</label>
							<div class="control">
								<input type="color" class="input" />
							</div>
						</div>

						<!-- Image Upload -->
						<div class="field">
							<label class="label">Upload Background Image</label>
							<div class="file has-name is-fullwidth">
								<label class="file-label">
									<input
										class="file-input"
										type="file"
										accept="image/*"
										on:change={(e) => (selectedImageFile = e.target.files[0])}
									/>
									<span class="file-cta">
										<span class="file-label">Choose an image…</span>
									</span>
									<span class="file-name">No image chosen</span>
								</label>
							</div>
						</div>
						<div class="control">
							<button class="button is-link is-fullwidth" on:click={setBackgroundImage}
								>Create Canvas with Background Image</button
							>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</section>

<section class="section">
	<div class="container">
		<div class="columns">
			<div class="column is-half">
				{#each canvases as canvas}
					<div class="box">
						<label class="checkbox">
							<input type="checkbox" bind:checked={canvas.sync} />
							Sync
						</label>

						<canvas class="canvas" id={canvas.id}></canvas>

						<button class="button is-link" on:click={() => exportCanvas(canvas.fabriccanvas)}
							>Export</button
						>
					</div>
				{/each}
			</div>
		</div>
	</div>
</section>

<style>
	/*canvas {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;

		-webkit-transform: scale(0.5);
		-webkit-transform-origin: 0 0;
		transform-origin: 0 0;
	} */

	.main-canvas {
		border: 1px solid #ccc;
		/* border-radius: 10px; */
	}
</style>
