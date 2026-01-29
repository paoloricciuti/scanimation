<script lang="ts">
	import '@fontsource/instrument-serif/400-italic.css';
	import '@fontsource/dm-mono/400.css';

	function base_64(file: File | undefined) {
		if (!file) return;
		return new Promise<string>((resolve, reject) => {
			const reader = new FileReader();
			reader.readAsDataURL(file);
			reader.onload = () => resolve(reader.result as string);
			reader.onerror = reject;
		});
	}

	async function files_to_base64(files: FileList | undefined) {
		if (!files) return [];
		const promises = Array.from(files).map((file) => base_64(file));
		const results = await Promise.all(promises);
		return results.filter((r): r is string => r !== undefined);
	}

	let files = $state<FileList | undefined>(undefined);
	let images = $derived(await files_to_base64(files));
	let x = $state(0);
	let y = $state(0);
	let column_size = $state(1);
	let amount = $derived(images.length);
	let main_el = $state<HTMLElement | null>(null);
	let dragging = $state(false);

	let snapped_x = $derived.by(() => {
		if (!main_el) return x;
		const rect = main_el.getBoundingClientRect();
		const image_left = rect.left + (rect.width - 320) / 2;
		const relative_x = x - image_left;
		const snapped = Math.round(relative_x / column_size) * column_size;
		return image_left + snapped;
	});

	let snapped_y = $derived(Math.round(y / column_size) * column_size);
</script>

<svelte:window
	onmousemove={(e) => {
		x = e.clientX;
		y = e.clientY;
	}}
/>

<svelte:head>
	<title>Scanimation - Optical Illusion Generator</title>
</svelte:head>

<div class="page">
	<header>
		<h1>Scanimation</h1>
		<p class="tagline">Optical illusion generator</p>
	</header>

	<section class="controls" class:has-images={images.length > 0}>
		<label
			class="file-drop"
			class:dragging
			ondragover={(e) => {
				e.preventDefault();
				dragging = true;
			}}
			ondragleave={() => (dragging = false)}
			ondrop={(e) => {
				e.preventDefault();
				dragging = false;
				if (e.dataTransfer?.files) {
					files = e.dataTransfer.files;
				}
			}}
		>
			<input type="file" bind:files multiple accept="image/*" />
			<span class="drop-content">
				<span class="drop-icon">
					<svg
						width="32"
						height="32"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
					>
						<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
						<polyline points="17 8 12 3 7 8" />
						<line x1="12" y1="3" x2="12" y2="15" />
					</svg>
				</span>
				<span class="drop-text">
					{#if images.length > 0}
						<strong>{images.length} frame{images.length !== 1 ? 's' : ''} loaded</strong>
						<small>Drop more or click to replace</small>
					{:else}
						<strong>Drop animation frames</strong>
						<small>or click to browse</small>
					{/if}
				</span>
			</span>
		</label>
		<button
			class="example-btn"
			onclick={async () => {
				const example = import.meta.glob('$lib/assets/svelte/*.png', {
					eager: true,
					import: 'default'
				});
				images = Object.values(example) as string[];
			}}>or try an example</button
		>
	</section>

	{#if images.length > 0}
		<div
			class="decoder"
			style:--columns={amount}
			style:--x={snapped_x}
			style:--y={snapped_y}
			style:--column-size="{column_size}px"
		></div>
		<main bind:this={main_el} style:--columns={amount} style:--column-size="{column_size}px">
			{#each images, i}
				<div class="frame" style:--i={i}>
					<img alt="Animation frame {i + 1}" src={images[i]} />
				</div>
			{/each}
		</main>
		<p class="hint">Move the decoder grid to reveal the animation</p>
		<div class="frames-preview">
			<span class="frames-label">Frames</span>
			<div class="frames-scroll">
				{#each images, i}
					<img alt="Frame {i + 1}" src={images[i]} class="frame-thumb" />
				{/each}
			</div>
		</div>
	{:else}
		<div class="empty-state">
			<div class="empty-illustration">
				<div class="scan-lines"></div>
			</div>
			<p>Upload sequential frames to create your scanimation</p>
		</div>
	{/if}

	<section class="controls bottom">
		<div class="slider-control">
			<div class="slider-header">
				<span class="slider-label">Strip width</span>
				<span class="slider-value">{column_size}px</span>
			</div>
			<input type="range" bind:value={column_size} min="1" max="10" class="styled-range" />
			<div class="slider-ticks">
				<span>1</span>
				<span>10</span>
			</div>
		</div>
	</section>

	<footer>
		<span>Move cursor over image to animate</span>
	</footer>
</div>

<style>
	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	:global(body) {
		background: #0a0a0b;
		color: #e8e4df;
		font-family: 'DM Mono', monospace;
		min-height: 100vh;
		overflow-x: hidden;
	}

	.page {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 3rem 2rem;
		position: relative;
		background:
			radial-gradient(ellipse at 50% 0%, rgba(60, 50, 40, 0.15) 0%, transparent 60%),
			repeating-linear-gradient(
				0deg,
				transparent,
				transparent 2px,
				rgba(255, 255, 255, 0.01) 2px,
				rgba(255, 255, 255, 0.01) 4px
			);
	}

	header {
		text-align: center;
		margin-bottom: 3rem;
	}

	h1 {
		font-family: 'Instrument Serif', serif;
		font-size: clamp(2.5rem, 8vw, 4.5rem);
		font-weight: 400;
		font-style: italic;
		letter-spacing: -0.02em;
		background: linear-gradient(135deg, #f5f0e8 0%, #c9b99a 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		margin-bottom: 0.5rem;
	}

	.tagline {
		font-size: 0.75rem;
		text-transform: uppercase;
		letter-spacing: 0.3em;
		color: #9a928a;
	}

	.controls {
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
		width: 100%;
		max-width: 360px;
		margin-bottom: 3rem;
		transition: margin-bottom 0.4s ease;
	}

	.controls.has-images {
		margin-bottom: 2rem;
	}

	.controls.bottom {
		margin-top: 3rem;
		margin-bottom: 0;
	}

	.file-drop {
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 2rem;
		border: 1px dashed #2a2825;
		border-radius: 2px;
		cursor: pointer;
		transition: all 0.2s ease;
		background: rgba(255, 255, 255, 0.01);
	}

	.file-drop:hover,
	.file-drop.dragging {
		border-color: #4a4540;
		background: rgba(255, 255, 255, 0.03);
	}

	.file-drop.dragging {
		border-color: #8b7355;
		box-shadow: 0 0 0 1px #8b7355;
	}

	.file-drop input {
		position: absolute;
		inset: 0;
		opacity: 0;
		cursor: pointer;
	}

	.drop-content {
		display: flex;
		align-items: center;
		gap: 1rem;
		pointer-events: none;
	}

	.drop-icon {
		color: #7a736a;
		transition: color 0.2s;
	}

	.file-drop:hover .drop-icon,
	.file-drop.dragging .drop-icon {
		color: #b89b7a;
	}

	.drop-text {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	.drop-text strong {
		font-size: 0.875rem;
		font-weight: 400;
		color: #e0d4c4;
	}

	.drop-text small {
		font-size: 0.7rem;
		color: #8a847a;
	}

	.slider-control {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.slider-header {
		display: flex;
		justify-content: space-between;
		align-items: baseline;
	}

	.slider-label {
		font-size: 0.7rem;
		text-transform: uppercase;
		letter-spacing: 0.15em;
		color: #9a928a;
	}

	.slider-value {
		font-size: 0.875rem;
		color: #e0d4c4;
		font-variant-numeric: tabular-nums;
	}

	.styled-range {
		-webkit-appearance: none;
		appearance: none;
		width: 100%;
		height: 2px;
		background: #2a2825;
		border-radius: 1px;
		cursor: pointer;
	}

	.styled-range::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 14px;
		height: 14px;
		background: #0a0a0b;
		border: 2px solid #8b7355;
		border-radius: 50%;
		cursor: grab;
		transition: all 0.15s ease;
	}

	.styled-range::-webkit-slider-thumb:hover {
		background: #1a1918;
		border-color: #c9b99a;
		transform: scale(1.1);
	}

	.styled-range::-webkit-slider-thumb:active {
		cursor: grabbing;
	}

	.styled-range::-moz-range-thumb {
		width: 14px;
		height: 14px;
		background: #0a0a0b;
		border: 2px solid #8b7355;
		border-radius: 50%;
		cursor: grab;
	}

	.slider-ticks {
		display: flex;
		justify-content: space-between;
		font-size: 0.6rem;
		color: #6a645a;
	}

	/* Decoder grid */
	.decoder {
		--space: calc(var(--column-size) * var(--columns));
		--hidden: calc(var(--space) - var(--column-size));
		--white: white;
		--black: black;
		--bg: repeating-linear-gradient(
			90deg,
			var(--white),
			var(--white) var(--hidden),
			var(--black) var(--hidden),
			var(--black) var(--space)
		);
		aspect-ratio: 1;
		position: fixed;
		z-index: 100;
		top: calc(var(--y) * 1px);
		left: calc(var(--x) * 1px);
		width: 30rem;
		transform: translate(-50%, -50%);
		pointer-events: none;
		background: black;
		mask: var(--bg) luminance;
		border-radius: 50%;
		box-shadow:
			0 0 60px 20px rgba(0, 0, 0, 0.8),
			inset 0 0 0 1px rgba(139, 115, 85, 0.1);
		opacity: 0;
		transition: opacity 0.2s ease;
	}

	.page:has(main:hover) .decoder {
		opacity: 1;
	}

	main {
		display: grid;
		place-items: center;
		grid-template-columns: 1fr;
		grid-template-rows: 1fr;
		padding: 2rem;
		background: rgba(255, 255, 255, 0.02);
		border: 1px solid #1a1815;
		border-radius: 2px;
		position: relative;
	}

	main::before {
		content: '';
		position: absolute;
		inset: -1px;
		border-radius: 2px;
		padding: 1px;
		background: linear-gradient(135deg, rgba(139, 115, 85, 0.2), transparent 50%);
		mask:
			linear-gradient(#fff 0 0) content-box,
			linear-gradient(#fff 0 0);
		mask-composite: exclude;
		-webkit-mask-composite: xor;
		pointer-events: none;
	}

	main::after {
		content: '';
		position: absolute;
		inset: -1rem -4rem;
	}

	.frame {
		--space: calc(var(--column-size) * var(--columns));
		--hidden: calc(var(--space) - var(--column-size));
		--white: black;
		--black: white;
		--bg: repeating-linear-gradient(
			90deg,
			var(--white),
			var(--white) var(--hidden),
			var(--black) var(--hidden),
			var(--black) var(--space)
		);
		grid-column: 1;
		grid-row: 1;
		width: 20rem;
		background: black;
		mask: var(--bg) luminance;
		mask-position: calc(var(--i) * var(--column-size));
	}

	.frame img {
		width: 100%;
		height: 100%;
		display: block;
	}

	.hint {
		margin-top: 1.5rem;
		font-size: 0.7rem;
		color: #8a847a;
		text-align: center;
		animation: pulse 2s ease-in-out infinite;
	}

	.frames-preview {
		margin-top: 2rem;
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		width: 100%;
		max-width: 360px;
	}

	.frames-label {
		font-size: 0.7rem;
		text-transform: uppercase;
		letter-spacing: 0.15em;
		color: #9a928a;
	}

	.frames-scroll {
		display: flex;
		gap: 0.5rem;
		overflow-x: auto;
		padding: 0.5rem;
		margin: -0.5rem;
		scrollbar-width: thin;
		scrollbar-color: #3a3530 transparent;
	}

	.frames-scroll::-webkit-scrollbar {
		height: 6px;
	}

	.frames-scroll::-webkit-scrollbar-track {
		background: transparent;
	}

	.frames-scroll::-webkit-scrollbar-thumb {
		background: #3a3530;
		border-radius: 3px;
	}

	.frames-scroll::-webkit-scrollbar-thumb:hover {
		background: #4a4540;
	}

	.frame-thumb {
		width: 3rem;
		height: 3rem;
		object-fit: cover;
		border-radius: 2px;
		border: 1px solid #2a2825;
		flex-shrink: 0;
	}

	@keyframes pulse {
		0%,
		100% {
			opacity: 0.5;
		}
		50% {
			opacity: 1;
		}
	}

	.empty-state {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
		padding: 4rem 2rem;
		color: #7a746a;
	}

	.empty-illustration {
		width: 200px;
		height: 140px;
		border: 1px dashed #2a2825;
		border-radius: 2px;
		position: relative;
		overflow: hidden;
	}

	.scan-lines {
		position: absolute;
		inset: 0;
		background: repeating-linear-gradient(
			90deg,
			transparent 0px,
			transparent 8px,
			rgba(139, 115, 85, 0.1) 8px,
			rgba(139, 115, 85, 0.1) 10px
		);
		animation: scan 3s linear infinite;
	}

	@keyframes scan {
		0% {
			transform: translateX(-10px);
		}
		100% {
			transform: translateX(0px);
		}
	}

	.empty-state p {
		font-size: 0.8rem;
		max-width: 240px;
		text-align: center;
		line-height: 1.6;
	}

	.example-btn {
		background: transparent;
		border: 1px solid #3a3530;
		color: #9a928a;
		padding: 0.75rem 1.5rem;
		font-family: 'DM Mono', monospace;
		font-size: 0.75rem;
		border-radius: 2px;
		cursor: pointer;
		transition: all 0.2s ease;
	}

	.example-btn:hover {
		border-color: #8b7355;
		color: #e0d4c4;
		background: rgba(139, 115, 85, 0.1);
	}

	footer {
		margin-top: auto;
		padding-top: 3rem;
		font-size: 0.65rem;
		color: #6a645a;
		text-transform: uppercase;
		letter-spacing: 0.2em;
	}
</style>
