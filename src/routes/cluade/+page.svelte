<script>
	import { onMount, onDestroy } from 'svelte';
	import { Canvas, Rect, Circle, Text, Image } from 'fabric';

	let canvas;
	let selectedObject = null;
	let color = '#000000';
	let fontSize = 20;
	let canvasElement;

	onMount(() => {
		canvas = new Canvas(canvasElement, {
			width: 600,
			height: 400,
			backgroundColor: '#ffffff'
		});

		canvas.on('selection:created', (e) => {
			selectedObject = e.selected[0];
		});

		canvas.on('selection:cleared', () => {
			selectedObject = null;
		});
	});

	onDestroy(() => {
		if (canvas) {
			canvas.dispose();
		}
	});

	function addRectangle() {
		if (!canvas) return;
		const rect = new Rect({
			left: 100,
			top: 100,
			fill: color,
			width: 100,
			height: 100
		});
		canvas.add(rect);
		canvas.setActiveObject(rect);
	}

	function addCircle() {
		if (!canvas) return;
		const circle = new Circle({
			left: 100,
			top: 100,
			fill: color,
			radius: 50
		});
		canvas.add(circle);
		canvas.setActiveObject(circle);
	}

	function addText() {
		if (!canvas) return;
		const text = new Text('Sample Text', {
			left: 100,
			top: 100,
			fill: color,
			fontSize
		});
		canvas.add(text);
		canvas.setActiveObject(text);
	}

	function addImage() {
		if (!canvas) return;
		const imgElement = document.createElement('img');
		imgElement.src = '/placeholder.jpg';
		imgElement.onload = () => {
			const image = new Image(imgElement, {
				left: 100,
				top: 100,
				scaleX: 0.5,
				scaleY: 0.5
			});
			canvas.add(image);
			canvas.setActiveObject(image);
		};
	}

	function deleteSelected() {
		if (!canvas || !selectedObject) return;
		canvas.remove(selectedObject);
		selectedObject = null;
	}

	$: if (selectedObject && color) {
		selectedObject.set('fill', color);
		canvas?.renderAll();
	}

	$: if (selectedObject && fontSize && selectedObject.type === 'text') {
		selectedObject.set('fontSize', parseInt(fontSize));
		canvas?.renderAll();
	}
</script>

<div class="canvas-editor">
	<div class="toolbar">
		<button on:click={addRectangle}> Add Rectangle </button>
		<button on:click={addCircle}> Add Circle </button>
		<button on:click={addText}> Add Text </button>
		<button on:click={addImage}> Add Image </button>
		<button on:click={deleteSelected} disabled={!selectedObject} class="delete-btn">
			Delete Selected
		</button>
	</div>

	<div class="controls">
		<div class="control-group">
			<label>
				Color:
				<input type="color" bind:value={color} />
			</label>
		</div>
		<div class="control-group">
			<label>
				Font Size:
				<input type="number" bind:value={fontSize} min="1" max="100" />
			</label>
		</div>
	</div>

	<div class="canvas-container">
		<canvas bind:this={canvasElement}></canvas>
	</div>
</div>

<style>
	.canvas-editor {
		width: 100%;
		max-width: 800px;
		margin: 0 auto;
		padding: 1rem;
	}

	.toolbar {
		display: flex;
		gap: 0.5rem;
		margin-bottom: 1rem;
		flex-wrap: wrap;
	}

	.controls {
		display: flex;
		gap: 1rem;
		margin-bottom: 1rem;
	}

	.control-group {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	button {
		padding: 0.5rem 1rem;
		background-color: #4a5568;
		color: white;
		border: none;
		border-radius: 0.25rem;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	button:hover {
		background-color: #2d3748;
	}

	button:disabled {
		background-color: #a0aec0;
		cursor: not-allowed;
	}

	.delete-btn {
		background-color: #e53e3e;
	}

	.delete-btn:hover {
		background-color: #c53030;
	}

	.canvas-container {
		border: 1px solid #e2e8f0;
		border-radius: 0.5rem;
		overflow: hidden;
	}

	input[type='color'] {
		width: 50px;
		height: 30px;
		padding: 0;
	}

	input[type='number'] {
		width: 60px;
		padding: 0.25rem;
	}

	label {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}
</style>
