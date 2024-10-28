<script>
	import { Canvas, Rect, Circle, Text, Image as FabricImage } from 'fabric';
	import { onMount } from 'svelte';
	import JSZip from 'jszip';

	let debug = false;

	let mainFabricCanvas;
	// {id: canvas1, fabriccanvas: fabricCanvasInstance, sync: bool,transformedStyleRatio,imageName}
	let canvases = []; // No predefined canvas IDs, dynamically add them
	//let = [];
	let inputText = '';
	let selectedImageFile = null; // For image uploads
	let selectedDocxFile = null; // For image uploads
	let selectedDocxFileZip = null;
	let useDebounce = true;

	let canvascontainer;
	let cwidth;
	let cheight;

	let fonts = ['Arial', 'Courier New', 'Georgia', 'Times New Roman', 'Verdana'];
	let selectedFont = 'Arial';
	let selectedColor = 'red';
	let selectedOpacity = 1;

	let style = '-webkit-transform: scale(1);';

	onMount(() => {
		mainFabricCanvas = new Canvas('mainFabricCanvas', {
			width: cwidth - 10,
			height: cheight - 10
		});

		// containersize = { width: ,height: };

		addrect();

		mainFabricCanvas.on('after:render', () => {
			if (!useDebounce) {
				syncCanvasObjects();
			} else {
				debouncedSyncCanvasObjects();
			}
		});

		//mainFabricCanvas.on('object:modified', syncCanvasObjects);
		//mainFabricCanvas.on('object:moving', syncCanvasObjects);
		//mainFabricCanvas.on('object:scaling', syncCanvasObjects);

		//// Object selection event
		//mainFabricCanvas.on('selection:created', function (e) {
		//	selectedObject = e.selected[0];
		//	syncCanvasObjects();
		//});

		//// Deselect event
		//mainFabricCanvas.on('selection:cleared', function () {
		//	selectedObject = null;
		//	syncCanvasObjects();
		//});
		////
		//mainFabricCanvas.on('selection:updated', function (e) {
		//	selectedObject = e.selected[0];
		//	syncCanvasObjects();
		//});
	});

	let fileInput;

	function handleDocxInput(event) {
		selectedDocxFile = event.target.files[0];
		if (selectedDocxFile && selectedDocxFile.name.endsWith('.docx')) {
			extractImages();
		} else {
			alert('Please upload a valid .docx file');
		}
	}

	async function exportDocx() {
		console.log('exporting');

		canvases.forEach((item) => {
			if (item.imageName !== undefined) {
				console.log(item.imageName);

				const exportedImageUrl = item.fabriccanvas.toDataURL({
					format: 'png',
					quality: 1, // Maximum quality for PNG
					multiplier: 1 // No scaling; export at full internal size
				});

				const binaryDataString = atob(exportedImageUrl.replace(/^data:image\/png;base64,/, ''));

				const bytes = new Uint8Array(binaryDataString.length);
				for (let i = 0; i < binaryDataString.length; i++) {
					bytes[i] = binaryDataString.charCodeAt(i);
				}

				selectedDocxFileZip.file(item.imageName, bytes, { binary: true });
			}
		});

		// Generate the new .docx file
		const newDocx = await selectedDocxFileZip.generateAsync({ type: 'blob' });

		// Create a download link for the new .docx file
		const downloadLink = document.createElement('a');
		downloadLink.href = URL.createObjectURL(newDocx);
		downloadLink.download = 'watermarked_docx.docx';
		downloadLink.click();
	}

	async function extractImages() {
		console.log('extracting images');

		const reader = new FileReader();

		reader.onload = async function (event) {
			const arrayBuffer = event.target.result;

			selectedDocxFileZip = new JSZip();
			await selectedDocxFileZip.loadAsync(arrayBuffer);

			const imageFiles = Object.keys(selectedDocxFileZip.files).filter((fileName) =>
				fileName.includes('media/')
			);
			console.log(imageFiles);

			for (let i = 0; i < imageFiles.length; i++) {
				const imageFile = selectedDocxFileZip.files[imageFiles[i]];

				const imageBlob = await imageFile.async('blob');
				setImageCanvas(URL.createObjectURL(imageBlob), imageFiles[i]);
			}
			syncCanvasObjects();
		};

		reader.readAsArrayBuffer(selectedDocxFile);
	}

	function addrect() {
		const rect = new Rect({
			left: 100,
			top: 100,
			fill: selectedColor,
			width: 50,
			height: 50
		});
		mainFabricCanvas.add(rect);
	}

	function addcircle() {
		const circle = new Circle({
			radius: 20,
			fill: selectedColor,
			left: 100,
			top: 100
		});
		mainFabricCanvas.add(circle);
	}

	function addText() {
		const text = new Text(inputText, {
			left: 100,
			top: 100,
			fontFamily: selectedFont,
			fill: 'black'
		});
		mainFabricCanvas.add(text);
	}

	function updateSelectionColor(newColor) {
		const activeObject = mainFabricCanvas.getActiveObject();

		console.log({ selectedColor });
		if (activeObject) {
			activeObject.set('fill', newColor);
			mainFabricCanvas.requestRenderAll();
		}
	}

	function updateSelectionOpacity(newOpacity) {
		const activeObject = mainFabricCanvas.getActiveObject();

		console.log({ selectedColor });
		if (activeObject) {
			activeObject.set('opacity', newOpacity);
			mainFabricCanvas.requestRenderAll();
		}
	}

	function updateSelectionFont(newFont) {
		const activeObject = mainFabricCanvas.getActiveObject();
		if (activeObject && activeObject.type === 'text') {
			activeObject.set('fontFamily', newFont);
			mainFabricCanvas.requestRenderAll();
		}
	}

	//opacity

	function deleteSelection() {
		mainFabricCanvas.remove(mainFabricCanvas.getActiveObject());
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
				setImageCanvas(event.target.result);
			};
			console.log(selectedImageFile);
			reader.readAsDataURL(selectedImageFile);
		}
	}

	function setImageCanvas(imageUrl, imageName = undefined) {
		const imgElement = new Image();
		imgElement.src = imageUrl; //imageBlob
		console.log({ imgElement });

		imgElement.onload = function () {
			// Dynamically generate a new canvas ID
			const newCanvasId = `canvas${canvases.length + 1}`;
			//const transformedStyleRatio = `-webkit-transform: scale(${(window.screen.width / imgElement.width) * (2 / 3)});`;
			const transformedStyleRatio = (window.screen.width / imgElement.width) * (2 / 3);
			console.log(transformedStyleRatio);
			canvases = [
				...canvases,
				{
					id: newCanvasId,
					sync: true,
					transformedStyleRatio: transformedStyleRatio,
					imageName: imageName
				}
			]; // Update canvases array

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
	}

	function setBgAll() {
		canvases.forEach((canvas) => {
			console.log('canvas: ', { canvas });
			canvas.set({ backgroundImage: canvas.fabriccanvas.canvasBGImage }); // this is setting background image when i click button
			canvas.requestRenderAll();
		});
	}

	function debounce(func, wait) {
		let timeout;
		return function (...args) {
			clearTimeout(timeout);
			timeout = setTimeout(() => func.apply(this, args), wait);
		};
	}

	const debouncedSyncCanvasObjects = debounce(syncCanvasObjects, 100);

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
					//canvases;

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
						on:change={handleDocxInput}
					/>
					<span class="file-cta">
						<span class="file-label">Choose a docx file…</span>
					</span>
					<span class="file-name">
						{selectedDocxFile ? selectedDocxFile.name : 'No file chosen'}
					</span>
				</label>
			</div>
		</div>

		<div class="field">
			<div class="control">
				<!-- <button class="button is-primary" on:click={processDocx}>Add Watermark</button> -->
				<button class="button is-primary" on:click={exportDocx}>Apply Changes</button>
			</div>
		</div>

		<a id="downloadLink" style="display: none;">Download Watermarked Docx</a>
	</div>
</section>

<section class="section">
	<div class="container">
		<div class="columns">
			<!-- Left Column: Main Canvas -->
			<div bind:clientWidth={cwidth} bind:clientHeight={cheight} class="column is-two-thirds">
				<!-- <div bind:this={canvascontainer} class="column is-two-thirds"> -->
				<!-- <div id="mainCanvasContainer" class="box"> -->
				<!-- {#if containersize} -->
				<!-- <canvas id="mainFabricCanvas" class="main-canvas" width={cwidth} height={cheight}></canvas> -->
				<canvas id="mainFabricCanvas" class="main-canvas"></canvas>

				<!-- width={`${window.screen.width * (2 / 3) - 100}px`} -->
				<!-- height={`${window.screen.height * (2 / 3)}px`} -->
				<!-- {/if} -->
				<!-- </div> -->
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
							<button class="button is-link is-fullwidth" on:click={addrect}>
								Add Rectangle
							</button>
							<button class="button is-primary is-fullwidth" on:click={addcircle}>
								Add Circle
							</button>
						</div>

						<div>
							<button class="button is-danger is-fullwidth" on:click={deleteSelection}>
								Delete
							</button>
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

						<!-- {#if mainFabricCanvas.getActiveObject()} -->
						<div>
							<label for="color" class="label">Pick Color</label>
							<div name="color" class="control">
								<input
									type="color"
									class="input"
									bind:value={selectedColor}
									on:input={(e) => updateSelectionColor(e.target.value)}
								/>
							</div>

							<!-- Opacity Slider -->
							<label for="opacity-slider" class="label">Set Opacity:</label>
							<input
								id="opacity-slider"
								type="range"
								min="0"
								max="1"
								step="0.01"
								bind:value={selectedOpacity}
								on:input={(e) => updateSelectionOpacity(e.target.value)}
							/>

							<!-- {#if selectedObject.type === 'textbox'} -->
							<div>
								<label for="font" class="label">Font family:</label>

								<div class="select is-link">
									<select
										bind:value={selectedFont}
										on:change={(e) => updateSelectionFont(e.target.value)}
										class="select"
									>
										{#each fonts as font}
											<option value={font}>{font}</option>
										{/each}
									</select>
								</div>
							</div>
							<!-- {/if} -->
						</div>
						<!-- {/if} -->

						<details>
							<summary> Advanced </summary>
							<div>
								<label>
									<input type="checkbox" bind:checked={useDebounce} />
									Enable Debouncing (improved perf)
								</label>
							</div>
						</details>

						{#if debug}
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
										<span class="file-name">
											{selectedImageFile ? selectedImageFile.name : 'hell'}</span
										>
									</label>
								</div>
							</div>

							<div class="control">
								<button class="button is-link is-fullwidth" on:click={setBackgroundImage}
									>Create Canvas with Background Image</button
								>
							</div>
						{/if}
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
						<!-- <canvas -->
						<!-- 	class="canvas" -->
						<!-- 	id={canvas.id} -->
						<!-- 	style={`-webkit-transform: scale(${canvas.transformedStyleRatio});`} -->
						<!-- ></canvas> -->

						<button class="button is-link" on:click={() => exportCanvas(canvas.fabriccanvas)}>
							Export
						</button>
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
		border-radius: 10px;
	}
</style>
