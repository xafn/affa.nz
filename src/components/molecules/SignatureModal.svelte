<script lang="ts">
	import { createEventDispatcher, onMount, tick } from 'svelte';
	import type { SignaturePayload, SignaturePoint, SignatureStroke } from '../../util/types';
	import { fly } from 'svelte/transition';
	import { cubicOut, quintOut } from 'svelte/easing';
	import { SIGNATURE_MESSAGE_MAX, SIGNATURE_NAME_MAX } from '../../util/constants';

	export let open = false;
	export let saving = false;
	export let errorMessage = '';
	export let signedInEmail = '';

	const dispatch = createEventDispatcher<{
		close: void;
		submit: SignaturePayload;
	}>();

	const CANVAS_WIDTH = 520;
	const CANVAS_HEIGHT = 280;
	const SIGNATURE_WIDTH = 5;
	const NAME_MAX = SIGNATURE_NAME_MAX;
	const MESSAGE_MAX = SIGNATURE_MESSAGE_MAX;

	let signatureName = '';
	let signatureMessage = '';
	let accentColor = '#000000';

	let canvasEl: HTMLCanvasElement | null = null;
	let drawing = false;
	let currentStroke: SignatureStroke | null = null;
	let strokes: SignatureStroke[] = [];
	let redoStack: SignatureStroke[] = [];

	$: hasSignature = strokes.length > 0;
	$: canSubmit = signatureName.trim().length > 0 && hasSignature && !saving;
	$: nameCount = signatureName.length;
	$: messageCount = signatureMessage.length;
	$: if (open) {
		void setupModal();
	}

	onMount(() => {
		refreshAccentColor();
		const media = window.matchMedia('(prefers-color-scheme: dark)');
		const onThemeChange = () => {
			refreshAccentColor();
			renderCanvas();
		};
		media.addEventListener('change', onThemeChange);

		return () => {
			media.removeEventListener('change', onThemeChange);
		};
	});

	async function setupModal() {
		refreshAccentColor();
		await tick();
		initCanvas();
	}

	function refreshAccentColor() {
		if (typeof window === 'undefined') return;
		const value = getComputedStyle(document.documentElement).getPropertyValue('--accent').trim();
		accentColor = value || '#000000';
	}

	function getPointFromEvent(event: PointerEvent): SignaturePoint {
		if (!canvasEl) {
			return { x: 0, y: 0 };
		}
		const rect = canvasEl.getBoundingClientRect();
		return {
			x: ((event.clientX - rect.left) / rect.width) * canvasEl.width,
			y: ((event.clientY - rect.top) / rect.height) * canvasEl.height
		};
	}

	function initCanvas() {
		if (!canvasEl) return;
		const ctx = canvasEl.getContext('2d');
		if (!ctx) return;
		ctx.lineCap = 'round';
		ctx.lineJoin = 'round';
		renderCanvas();
	}

	function renderSingleStroke(stroke: SignatureStroke, context: CanvasRenderingContext2D) {
		if (!stroke || !stroke.points || stroke.points.length === 0) return;

		context.save();
		context.globalCompositeOperation = 'source-over';
		context.strokeStyle = accentColor;
		context.lineWidth = stroke.width;
		context.beginPath();

		if (stroke.points.length === 1) {
			const point = stroke.points[0];
			context.arc(point.x, point.y, stroke.width / 2, 0, Math.PI * 2);
			context.fillStyle = accentColor;
			context.fill();
		} else {
			context.moveTo(stroke.points[0].x, stroke.points[0].y);
			for (let i = 1; i < stroke.points.length; i += 1) {
				context.lineTo(stroke.points[i].x, stroke.points[i].y);
			}
			context.stroke();
		}

		context.restore();
	}

	function renderCanvas() {
		if (!canvasEl) return;
		const ctx = canvasEl.getContext('2d');
		if (!ctx) return;

		ctx.clearRect(0, 0, canvasEl.width, canvasEl.height);
		strokes.forEach((stroke) => renderSingleStroke(stroke, ctx));
		if (currentStroke) {
			renderSingleStroke(currentStroke, ctx);
		}
	}

	function startStroke(event: PointerEvent) {
		if (event.button !== 0 || !canvasEl) return;
		drawing = true;
		const point = getPointFromEvent(event);
		currentStroke = {
			width: SIGNATURE_WIDTH,
			points: [point]
		};
		canvasEl.setPointerCapture(event.pointerId);
		renderCanvas();
	}

	function continueStroke(event: PointerEvent) {
		if (!drawing || !currentStroke) return;
		const point = getPointFromEvent(event);
		currentStroke.points = [...currentStroke.points, point];
		renderCanvas();
	}

	function endStroke() {
		if (!drawing) return;
		drawing = false;
		if (currentStroke && currentStroke.points.length > 0) {
			strokes = [...strokes, currentStroke];
			redoStack = [];
		}
		currentStroke = null;
		renderCanvas();
	}

	function undoStroke() {
		if (!strokes.length) return;
		const next = [...strokes];
		const removed = next.pop();
		if (removed) {
			redoStack = [...redoStack, removed];
		}
		strokes = next;
		renderCanvas();
	}

	function redoStroke() {
		if (!redoStack.length) return;
		const nextRedo = [...redoStack];
		const restored = nextRedo.pop();
		if (!restored) return;
		redoStack = nextRedo;
		strokes = [...strokes, restored];
		renderCanvas();
	}

	function clearCanvas() {
		strokes = [];
		redoStack = [];
		currentStroke = null;
		renderCanvas();
	}

	function resetForm() {
		signatureName = '';
		signatureMessage = '';
		clearCanvas();
	}

	function close() {
		resetForm();
		dispatch('close');
	}

	function onBackdropKeydown(event: KeyboardEvent) {
		if (event.key === 'Escape') {
			event.preventDefault();
			close();
		}
	}

	function submitSignature() {
		if (!canSubmit || !canvasEl) return;
		dispatch('submit', {
			name: signatureName.trim(),
			message: signatureMessage.trim() || null,
			signature_data: canvasEl.toDataURL('image/png')
		});
	}
</script>

{#if open}
	<div
		class="modal-backdrop"
		on:click|self={close}
		on:keydown={onBackdropKeydown}
		role="button"
		tabindex="0"
		aria-label="Close signature modal"
		in:fly={{ y: 50, easing: quintOut, duration: 750 }}
		out:fly={{ y: 50, easing: cubicOut, duration: 300 }}
	>
		<section
			class="modal"
			in:fly={{ y: 50, easing: quintOut, duration: 750 }}
			out:fly={{ y: 50, easing: cubicOut, duration: 300 }}
		>
			<header class="modal-head">
				<h3>Add your signature</h3>
				<button type="button" class="close" on:click={close} aria-label="Close">
					<span class="material-symbols-outlined" aria-hidden="true">close</span>
				</button>
			</header>
			<p class="modal-subtext">
				Add your name and signature, and a short message if you want! Keep things proper, I can
				remove anything for any reason. Thank you!
			</p>
			{#if signedInEmail}
				<p class="modal-account">Signed in as {signedInEmail}</p>
			{/if}

			<div class="field-grid">
				<label>
					<div class="label-row">
						<h6>Name *</h6>
						<span class="char-count" aria-live="polite">{nameCount}/{NAME_MAX}</span>
					</div>
					<input
						type="text"
						maxlength={NAME_MAX}
						placeholder="afn zmn"
						bind:value={signatureName}
						required
					/>
				</label>

				<label>
					<div class="label-row">
						<h6>Message</h6>
						<span class="char-count" aria-live="polite">{messageCount}/{MESSAGE_MAX}</span>
					</div>
					<textarea
						rows="2"
						maxlength={MESSAGE_MAX}
						placeholder="you're so cool and hot affan"
						bind:value={signatureMessage}
					/>
				</label>
			</div>

			<div class="canvas-wrap">
				<canvas
					bind:this={canvasEl}
					width={CANVAS_WIDTH}
					height={CANVAS_HEIGHT}
					on:pointerdown={startStroke}
					on:pointermove={continueStroke}
					on:pointerup={endStroke}
					on:pointerleave={endStroke}
					on:pointercancel={endStroke}
				/>

				<div class="tools" aria-label="Canvas actions">
					<button
						type="button"
						class="icon-btn"
						on:click={undoStroke}
						disabled={!strokes.length}
						aria-label="Undo"
						title="Undo"
					>
						<span class="material-symbols-outlined" aria-hidden="true">undo</span>
					</button>
					<button
						type="button"
						class="icon-btn"
						on:click={redoStroke}
						disabled={!redoStack.length}
						aria-label="Redo"
						title="Redo"
					>
						<span class="material-symbols-outlined" aria-hidden="true">redo</span>
					</button>
					<button
						type="button"
						class="icon-btn"
						on:click={clearCanvas}
						disabled={!strokes.length}
						aria-label="Clear signature"
						title="Clear"
					>
						<span class="material-symbols-outlined" aria-hidden="true">delete</span>
					</button>
				</div>
			</div>

			{#if errorMessage}
				<p class="error">{errorMessage}</p>
			{/if}

			<div class="modal-actions">
				<button type="button" class="ghost" on:click={close}>Cancel</button>
				<button type="button" class="primary" on:click={submitSignature} disabled={!canSubmit}>
					{saving ? 'Saving…' : 'Place!'}
				</button>
			</div>
		</section>
	</div>
{/if}

<style lang="scss">
	@import url('https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined');

	.modal-backdrop {
		background-blend-mode: overlay;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: fixed;
		left: 0;
		top: 50%;
		transform: translateY(-50%);
		height: 200vh;
		width: 100vw;
		z-index: 20;
		cursor: pointer;
		user-select: none;
		background-color: var(--elevation-six);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);

		@media (max-width: 768px) {
			padding: 0.4rem;
		}
	}

	.modal {
		width: min(580px, 100%);
		max-height: 95vh;
		overflow: auto;
		padding: 1.3rem;
		border-radius: 20px;
		border: 1px solid var(--elevation-four);
		background-color: var(--bg-color);
		display: grid;
		gap: 1rem;
		box-shadow: 0 30px 70px rgba(0, 0, 0, 0.35);

		@media (max-width: 768px) {
			width: min(85%, 440px);
			padding: 0.8rem;
			border-radius: 14px;
			gap: 0.65rem;
		}
	}

	.modal-head {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 0.1rem 0 1.15rem;
		border-bottom: 1px solid var(--elevation-four);
		gap: 1rem;

		h3 {
			line-height: 1;
		}

		@media (max-width: 768px) {
			padding-bottom: 0.45rem;

			h3 {
				font-size: 1.1rem;
			}
		}
	}

	.modal-subtext {
		font-size: 0.88rem;
		line-height: 1.35;
		color: var(--text-secondary);

		@media (max-width: 768px) {
			font-size: 0.76rem;
			line-height: 1.25;
		}
	}

	.modal-account {
		font-size: 0.78rem;
		font-family: var(--font-two);
		color: var(--accent);
		margin-top: -0.35rem;
	}

	.field-grid {
		display: grid;
		grid-template-columns: 1fr;
		gap: 1rem;
		margin-bottom: 0.35rem;

		@media (max-width: 768px) {
			gap: 0.8rem;
		}

		@media (min-width: 700px) {
			grid-template-columns: 1fr 1fr;
		}

		label:last-child {
			@media (min-width: 700px) {
				grid-column: span 2;
			}
		}

		label + label {
			margin-top: 0.1rem;
		}
	}

	.close {
		font-family: 'Material Symbols Outlined';
		font-size: 2.1rem;
		line-height: 1;
		font-variation-settings:
			'FILL' 0,
			'wght' 400,
			'GRAD' 0,
			'opsz' 24;
		padding: 0;
		border: 0;
		background: transparent;
		color: var(--text-primary);
		cursor: pointer;
		transition:
			color 0.2s var(--bezier-one),
			transform 0.2s var(--bezier-one);
		display: inline-flex;
		align-items: center;
		justify-content: center;
		margin-top: -0.18rem;

		&:hover {
			color: var(--accent);
			transform: scale(1.05);
		}

		@media (max-width: 768px) {
			font-size: 1.75rem;
			margin-top: -0.1rem;
		}
	}

	label {
		display: grid;
		gap: 0.35rem;

		@media (max-width: 768px) {
			gap: 0.2rem;
		}

		.label-row {
			display: flex;
			align-items: center;
			justify-content: space-between;
			gap: 0.5rem;
		}

		h6 {
			font-size: 0.78rem;
			font-family: var(--font-two);
			text-transform: none;
			letter-spacing: 0;
			margin: 0;
			padding: 0;
			background: transparent;
		}

		.char-count {
			font-family: var(--font-two);
			font-size: 0.68rem;
			letter-spacing: 0.01em;
			color: var(--text-secondary);
		}
	}

	input,
	textarea {
		width: 100%;
		font-size: 0.95rem;
		font-family: var(--font-one);
		padding: 0.75rem;
		border-radius: 10px;
		border: 1px solid var(--elevation-four);
		background: var(--elevation-one);
		color: var(--text-primary);
		transition:
			border-color 0.15s ease,
			box-shadow 0.15s ease;

		&:focus-visible {
			outline: none;
			border-color: var(--accent);
			box-shadow: 0 0 0 2px var(--accent-opacity);
		}

		@media (max-width: 768px) {
			padding: 0.55rem;
			font-size: 0.85rem;
		}
	}

	textarea {
		resize: none;
	}

	.tools {
		display: flex;
		flex-wrap: wrap;
		justify-content: flex-end;
		gap: 0.5rem;
		position: absolute;
		right: 0.75rem;
		bottom: 0.75rem;
		padding: 0.45rem 0.65rem;
		border-radius: 14px;
		background: color-mix(in srgb, var(--bg-color) 92%, transparent);
		border: 1px solid var(--elevation-four);
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
		z-index: 2;

		@media (max-width: 768px) {
			right: 0.5rem;
			bottom: 0.5rem;
			padding: 0.34rem 0.5rem;
			gap: 0.38rem;
		}

		button {
			width: auto;
			height: auto;
			padding: 0;
			display: inline-flex;
			align-items: center;
			justify-content: center;
			border: 0;
			background: transparent;
			color: var(--text-primary);
			cursor: pointer;
			line-height: 1;
			border-radius: 0;
			transition:
				color 0.2s var(--bezier-one),
				opacity 0.2s var(--bezier-one);

			&:hover:not(:disabled) {
				color: var(--accent);
			}

			&:disabled {
				opacity: 0.25;
				cursor: not-allowed;
			}

			span {
				font-family: 'Material Symbols Outlined';
				font-size: 1.35rem;
				font-variation-settings:
					'FILL' 0,
					'wght' 400,
					'GRAD' 0,
					'opsz' 24;
				line-height: 1;
			}

			@media (max-width: 768px) {
				span {
					font-size: 1.15rem;
				}
			}
		}
	}

	.canvas-wrap {
		position: relative;
		margin-top: 0.2rem;
		border-radius: 12px;
		border: 1px solid var(--elevation-four);
		background: var(--elevation-one);
		overflow: hidden;
		aspect-ratio: 520 / 280;

		&::before {
			content: '';
			position: absolute;
			inset: 0;
			background-image:
				linear-gradient(to right, var(--elevation-four) 1px, transparent 1px),
				linear-gradient(to bottom, var(--elevation-four) 1px, transparent 1px);
			background-size: 22px 22px;
			opacity: 0.42;
			pointer-events: none;
		}

		canvas {
			position: relative;
			z-index: 1;
			display: block;
			width: 100%;
			height: 100%;
			touch-action: none;
			cursor: crosshair;

			@media (max-width: 768px) {
				height: 100%;
			}
		}
	}

	.modal-actions {
		display: flex;
		justify-content: flex-end;
		gap: 0.6rem;

		@media (max-width: 768px) {
			gap: 0.4rem;
		}

		button {
			font-family: var(--font-two);
			padding: 0.55rem 0.85rem;
			border-radius: 10px;
			border: 1px solid var(--elevation-four);
			cursor: pointer;

			@media (max-width: 768px) {
				padding: 0.45rem 0.62rem;
				font-size: 0.75rem;
			}
		}

		.ghost {
			background: var(--elevation-one);
			color: var(--text-primary);

			&:hover {
				filter: brightness(1.08);
			}
		}

		.primary {
			background: var(--accent);
			color: var(--bg-color);

			&:hover:not(:disabled) {
				filter: brightness(1.06);
			}

			&:disabled {
				opacity: 0.5;
				cursor: not-allowed;
			}
		}
	}
</style>
