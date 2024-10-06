<script>
	import { Canvas, Rect, Circle, Text, Image as FabricImage } from 'fabric';
	import { onMount } from 'svelte';
	import JSZip from 'jszip';

	let mainFabricCanvas;
	let canvases = [];
	let selectedDocxFile = null; // For .docx uploads
	let zip = new JSZip(); // Initialize JSZip

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

	// Extract images from the .docx file
	async function extractImagesFromDocx() {
		if (selectedDocxFile) {
			const zipFile = await JSZip.loadAsync(selectedDocxFile);
			zip = zipFile; // Store zip for later repackaging

			const imagePaths = [];
			zip.forEach((relativePath, file) => {
				if (relativePath.startsWith('word/media/') && file.name.match(/\.(png|jpg|jpeg)$/)) {
					imagePaths.push(relativePath);
				}
			});

			// Load images into canvases
			imagePaths.forEach(async (path, index) => {
				const imageBlob = await zip.file(path).async('blob');
				const imgElement = new Image();
				imgElement.src = URL.createObjectURL(imageBlob);
				imgElement.onload = function () {
					const newCanvasId = `canvas${canvases.length + 1}`;
					canvases = [...canvases, { id: newCanvasId, sync: true }]; // Update canvases array

					setTimeout(() => {
						const newCanvas = new Canvas(newCanvasId);
						const canvasBGImage = new FabricImage(imgElement);
						newCanvas.set({ backgroundImage: canvasBGImage });
						newCanvas.setDimensions({ width: imgElement.width, height: imgElement.height });
						newCanvas.requestRenderAll();

						findCanvasById(newCanvasId).fabriccanvas = newCanvas;
					}, 0);
				};
			});
		}
	}

	const findCanvasById = (id) => {
		return canvases.find((canvas) => canvas.id === id);
	};

	// Sync canvas objects across all canvases
	function syncCanvasObjects() {
		const objectsJSON = mainFabricCanvas.toJSON();
		canvases.forEach((canvasItem) => {
			let canvas = canvasItem.fabriccanvas;
			if (canvas !== mainFabricCanvas && canvasItem.sync) {
				canvas.loadFromJSON(objectsJSON, () => {
					canvas.requestRenderAll();
				});
			}
		});
	}

	// Download the modified .docx file
	async function downloadModifiedDocx() {
		// Iterate over each canvas and export modified images
		for (let canvasItem of canvases) {
			const canvas = canvasItem.fabriccanvas;
			const dataUrl = canvas.toDataURL(); // Export modified canvas as image
			const imageBlob = await (await fetch(dataUrl)).blob();

			// Update the images in the .docx
			const filePath = `word/media/image${canvases.indexOf(canvasItem) + 1}.png`; // Assuming image numbering
			zip.file(filePath, imageBlob);
		}

		// Generate the new .docx file
		const modifiedDocxBlob = await zip.generateAsync({ type: 'blob' });
		const modifiedDocxUrl = URL.createObjectURL(modifiedDocxBlob);

		// Create download link
		const a = document.createElement('a');
		a.href = modifiedDocxUrl;
		a.download = 'modified_document.docx';
		a.click();
	}
</script>

<div>
	<canvas id="mainFabricCanvas" width="500" height="300"></canvas>
	<button on:click={addrect} class="button">Add Rectangle</button>
	<button on:click={syncCanvasObjects} class="button">Sync Canvases</button>

	<!-- .docx upload section -->
	<input type="file" accept=".docx" on:change={(e) => (selectedDocxFile = e.target.files[0])} />
	<button on:click={extractImagesFromDocx}>Load Images from .docx</button>

	<!-- Download modified .docx -->
	<button on:click={downloadModifiedDocx}>Download Modified .docx</button>

	{#each canvases as id}
		<div>
			<br />
			<br />
			sync: {id.sync}
			<input type="checkbox" bind:checked={id.sync} />
			<canvas id={id.id}></canvas>
		</div>
	{/each}
</div>

<style>
	canvas {
		border: 1px solid black;
	}
</style>
