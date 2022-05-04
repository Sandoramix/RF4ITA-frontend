<script>
	import { createEventDispatcher, onMount } from "svelte";

	const dispatch = createEventDispatcher();

	var drawToolsContainer;

	var canDraw = false;

	var drawingCanvas,
		ctx,
		drawingLines = [],
		drawCorrectionX = 18,
		drawCorrectionY = 160,
		isDrawing = false,
		drawNewPosX = 0,
		drawNewPosY = 0;

	function startDrawing(ev) {
		ev.stopPropagation();
		isDrawing = true;
		drawingLines.push([]);
	}

	function addNewLine() {
		drawingLines[drawingLines.length - 1].push({ x: drawNewPosX, y: drawNewPosY });
	}

	function drawAllLines() {
		ctx.beginPath();
		drawingLines.forEach((lines) => {
			ctx.moveTo(lines[0].x, lines[0].y);
			lines.forEach((line) => {
				ctx.lineTo(line.x, line.y);
			});
		});
		ctx.stroke();
	}

	function drawLine(ev) {
		ev.stopPropagation();
		ev.preventDefault();
		if (!isDrawing) return;

		ctx.lineJoin = ctx.lineCap = "round";

		if (!ev.changedTouches) {
			drawNewPosX = ev.layerX;
			drawNewPosY = ev.layerY;
		} else {
			drawNewPosX = ev.changedTouches[0].pageX - drawCorrectionX;
			drawNewPosY = ev.changedTouches[0].pageY - drawCorrectionY;
		}
		addNewLine();
		drawAllLines();
		ctx.stroke();
	}

	function drawToggle(ev) {
		ev.stopPropagation();
		isDrawing = false;
		addNewLine();
	}

	onMount(() => {
		drawToolsContainer = document.querySelector(`#draw-tools`);

		ctx = drawingCanvas.getContext("2d");
		ctx.lineWidth = 1;

		drawingCanvas.width = 600;
		drawingCanvas.height = 600;
	});

	const onPenClick = (ev) => {
		draw_tools.pen.classList.toggle(`active`);
		canDraw = !canDraw;
		dispatch(`map_toggle`);
		if (!canDraw) {
			drawingCanvas.classList.remove(`red-border`);
			drawingCanvas.style.display = drawToolsContainer.style.display = `none`;

			return clearCanvas(undefined);
		}
		drawToolsContainer.style.display = `flex`;
		drawingCanvas.classList.add(`red-border`);
		drawingCanvas.style.display = `block`;
	};
	const onSizeClick = (ev, mclick) => {
		ev.preventDefault();
		ev.stopPropagation();

		let currentPx = draw_tools.size.textContent;
		if (mclick == 1) {
			draw_tools.size.textContent = currentPx = currentPx === `1px` ? `2px` : currentPx === `2px` ? `5px` : currentPx === `3px` ? `5px` : `1px`;
		} else {
			draw_tools.size.textContent = currentPx = currentPx === `5px` ? `3px` : currentPx === `3px` ? `2px` : currentPx === `2px` ? `1px` : `5px`;
		}

		ctx.lineWidth = parseInt(currentPx);
		clearCanvas(false);
	};
	const onColorClick = (ev) => {
		let currentColor = window.getComputedStyle(draw_tools.color).backgroundColor;
		currentColor = currentColor === `rgb(0, 0, 0)` ? `rgb(255, 0, 0)` : `rgb(0, 0, 0)`;
		draw_tools.color.style.backgroundColor = currentColor;
		ctx.strokeStyle = currentColor;
		clearCanvas(false);
	};
	const onClearClick = (ev) => {
		ev.stopPropagation();
		clearCanvas(undefined);
	};

	const clearCanvas = (ev) => {
		ev = ev === undefined ? true : ev;
		ctx.clearRect(0, 0, drawingCanvas.width, drawingCanvas.height);
		!0 === ev && (drawingLines = []);
		isDrawing = false;
		drawAllLines();
	};
	var draw_tools = {
		color: null,
		clear: null,
		size: null,
		pen: null,
	};
</script>

<div id="draw-tools__container">
	<div id="draw-tools" style="display: none;">
		<div bind:this={draw_tools.color} class="tool tool-color " style="background-color: black;" on:click={onColorClick} />
		<div bind:this={draw_tools.clear} class="tool tool-clear" on:click={onClearClick}>X</div>
		<div bind:this={draw_tools.size} class="tool tool-size" on:click={(ev) => onSizeClick(ev, 1)} on:contextmenu={(ev) => onSizeClick(ev, 2)}>
			1px
		</div>
	</div>
	<div bind:this={draw_tools.pen} class="tool tool-pen" on:click={onPenClick}>✎</div>
</div>
<canvas
	bind:this={drawingCanvas}
	id="draw-canvas"
	style="display: none;"
	on:mousedown={startDrawing}
	on:mousemove={drawLine}
	on:mouseup={drawToggle}
	on:drag={drawToggle}
	on:touchstart={startDrawing}
	on:touchmove={drawLine}
	on:click={(ev) => ev.stopPropagation()}
/>

<style>
	#draw-canvas {
		z-index: 999;
		top: 0;
		left: 0;
		position: absolute;
		width: 600px;
		height: 600px;
		background-color: transparent;
	}

	#draw-tools__container {
		z-index: 1000;
		position: absolute;
		bottom: 20px;
		left: 12px;
	}

	#draw-tools__container,
	#draw-tools {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	#draw-tools {
		justify-content: end;
	}
	.tool {
		width: 30px;
		height: 30px;
		border: 1px solid var(--contrast-dark-color);
		line-height: 26px;
		text-align: center;
		font-weight: 600;
		font-size: 20px;

		user-select: none;
		padding: 0.125rem;
		background-color: #ffffffec;
		color: var(--accent-color);
	}

	.tool-size {
		font-size: 15px;
	}

	:global(.tool-pen.active) {
		background-color: var(--contrast-color) !important;
		color: red !important;
	}
	.tool-clear {
		color: red !important;
		font-weight: bold;
	}
	.tool-clear:hover {
		background-color: #ffbbbb !important;
	}

	.tool:hover {
		cursor: pointer;
		border: 1px solid var(--orange);
		background-color: var(--primary-color);
		color: var(--accent-dark-color);
	}
</style>
