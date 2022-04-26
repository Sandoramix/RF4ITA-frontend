<script lang="ts">
	import { createEventDispatcher, onMount } from "svelte";

	const dispatch = createEventDispatcher();

	var drawToolsContainer, drawColorBtn, drawClearBtn, drawLineSizeBtn, drawPenBtn;

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
		drawColorBtn = document.querySelector(`#draw-tools__color`);
		drawClearBtn = document.querySelector(`#draw-tools__clear`);
		drawLineSizeBtn = document.querySelector(`#draw-tools__px`);
		drawPenBtn = document.querySelector(`#draw-tools__pen`);

		drawPenBtn.addEventListener(`click`, (ev) => {
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
		});
		drawLineSizeBtn.addEventListener(`click`, (ev) => {
			ev.stopPropagation();
			let currentPx = drawLineSizeBtn.textContent;
			drawLineSizeBtn.textContent = currentPx = currentPx === `1px` ? `2px` : currentPx === `2px` ? `3px` : `1px`;
			ctx.lineWidth = parseInt(currentPx);
			clearCanvas(false);
		});
		drawClearBtn.addEventListener(`click`, (ev) => {
			ev.stopPropagation();
			clearCanvas(undefined);
		});
		drawColorBtn.addEventListener(`click`, () => {
			let currentColor = window.getComputedStyle(drawColorBtn).backgroundColor;
			currentColor = currentColor === `rgb(0, 0, 0)` ? `rgb(255, 0, 0)` : `rgb(0, 0, 0)`;
			drawColorBtn.style.backgroundColor = currentColor;
			ctx.strokeStyle = currentColor;
			clearCanvas(false);
		});

		drawingCanvas = document.querySelector("#draw-canvas");
		ctx = drawingCanvas.getContext("2d");
		ctx.lineWidth = 1;

		drawingCanvas.width = 650;
		drawingCanvas.height = 600;

		drawingCanvas.addEventListener(`mousedown`, startDrawing, false);
		drawingCanvas.addEventListener(`mousemove`, drawLine, false);
		drawingCanvas.addEventListener(`mouseup`, drawToggle, false);
		drawingCanvas.addEventListener(`drag`, drawToggle, false);
		drawingCanvas.addEventListener(`touchstart`, startDrawing, false);
		drawingCanvas.addEventListener(`touchmove`, drawLine, false);
		drawingCanvas.addEventListener(`mouseover`, () => {});
		drawingCanvas.addEventListener(`click`, (ev) => {
			ev.stopPropagation();
		});
	});

	function clearCanvas(ev) {
		ev = ev === undefined ? true : ev;
		ctx.clearRect(0, 0, drawingCanvas.width, drawingCanvas.height);
		!0 === ev && (drawingLines = []);
		isDrawing = false;
		drawAllLines();
	}
</script>

<div id="draw-tools__container">
	<div id="draw-tools" style="display: none;">
		<div id="draw-tools__color" style="background-color: black;" />
		<div id="draw-tools__clear">X</div>
		<div id="draw-tools__px">1px</div>
	</div>
	<div id="draw-tools__pen">✎</div>
</div>
<canvas id="draw-canvas" style="display: none;" />

<style>
	#draw-canvas {
		z-index: 999;
		top: 0;
		left: 0;
		position: absolute;
		width: 650px;
		height: 600px;
		background-color: transparent;
	}

	#draw-tools__container {
		z-index: 1000;
		position: absolute;
		bottom: 10px;
		left: 10px;
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

	#draw-tools__pen,
	#draw-tools__clear,
	#draw-tools__color,
	#draw-tools__px {
		width: 30px;
		height: 30px;
		border: 1px solid black;
		line-height: 26px;
		text-align: center;
		font-weight: 600;
	}

	#draw-tools__pen:hover,
	#draw-tools__clear:hover,
	#draw-tools__color:hover,
	#draw-tools__px:hover {
		cursor: pointer;
		border: 1px solid blue;
	}
</style>
