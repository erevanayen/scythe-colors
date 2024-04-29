<script lang="ts">
	import { onMount } from 'svelte';

	import designSwirl from '$lib/assets/pattern-noise.svg?raw';
	import designSpiral from '$lib/assets/pattern-spiral.svg?raw';
	import designTest from '$lib/assets/pattern-test.svg?raw';
	import desinClassic from '$lib/assets/pattern-classic.svg?raw';
	import crossIcon from '$lib/assets/icons/ic-cross.svg';

	type Color = {
		hex: string;
		picked: boolean;
	};

	type Design = {
		name: string;
		svg: HTMLElement | null;
		svgUrl: string;
		picked: boolean;
		hoodie: boolean;
		allPaths?: SVGPathElement[];
		pathsWithFill?: SVGPathElement[];
		pathsWithOutline?: SVGPathElement[];
	};

	type Result = {
		name: string;
		svg: HTMLElement;
	};

	const version = '1.0.0';
	const threadColorCap = 3;

	let threadColors: Color[] = [
		{ hex: '#000000', picked: false },
		{ hex: '#fb00ff', picked: false },
		{ hex: '#00f0e1', picked: false },
		{ hex: '#fff900', picked: false },
		{ hex: '#bcbcbc', picked: false },
		{ hex: '#150be0', picked: false },
		{ hex: '#360063', picked: false },
		{ hex: '#c30be0', picked: false },
		{ hex: '#f4b53f', picked: false },
		{ hex: '#890512', picked: false },
		{ hex: '#053827', picked: false },
		{ hex: '#e8bc5f', picked: false },
		{ hex: '#fcd202', picked: false },
		{ hex: '#e5e5e5', picked: false }
	];
	let fabricColors: Color[] = [
		{ hex: '#f1f1f1', picked: true },
		{ hex: '#fcefba', picked: false },
		{ hex: '#2c3d21', picked: false },
		// { hex: '#fcbfb5', picked: false },
		// { hex: '#e2b15f', picked: false },
		// { hex: '#87a5a3', picked: false },
		{ hex: '#000000', picked: false }
	];

	let designs: Design[] = [
		// { name: 'test', svg: null, svgUrl: designTest, picked: false },
		{ name: 'classic', svg: null, svgUrl: desinClassic, picked: false, hoodie: false },
		{ name: 'swirl', svg: null, svgUrl: designSwirl, picked: false, hoodie: false },
		{ name: 'spiral', svg: null, svgUrl: designSpiral, picked: false, hoodie: true }
	];
	let pickedDesign = designs[0];
	let initFlag = false;

	let pickedColors = 0;
	let totalPaths = 0; // pathsWithFill + pathsWithOutline
	$: varAmount = pickedColors ** totalPaths;

	let results: Result[] = [];

	onMount(() => {
		// initialize all designs with the svg element
		designs.forEach((design) => {
			design.svg = parseSvg(design.svgUrl);

			// process the svg and separate the paths
			separatePaths(design);
		});

		// set the first design as picked
		pickDesign(designs[0]);

		// allow rendering of designs and buttons
		initFlag = true;
	});

	// creates a svg element from a string
	function parseSvg(svg: string): HTMLElement {
		const parser = new DOMParser();
		const svgParsed = parser.parseFromString(svg, 'image/svg+xml').documentElement;

		return svgParsed;
	}

	function pickDesign(design: Design) {
		// set all designs to unpicked
		designs.forEach((d) => (d.picked = false));

		design.picked = true;
		pickedDesign = design;
		designs = [...designs];

		// set the total amount of paths
		if (design.pathsWithFill === undefined || design.pathsWithOutline === undefined) return;
		totalPaths = design.pathsWithFill.length + design.pathsWithOutline.length;
	}

	function separatePaths(design: Design) {
		if (design.svg === null) return;
		// get all paths with fill and paths with outline
		const allPaths = Array.from(design.svg.querySelectorAll('path'));
		const pathsWithFill = allPaths.filter((path) => path.style.fill !== 'none');
		const pathsWithOutline = allPaths.filter((path) => path.style.stroke);

		// set the paths
		design.allPaths = allPaths;
		design.pathsWithFill = pathsWithFill;
		design.pathsWithOutline = pathsWithOutline;
	}

	function clickThreadColor(color: Color) {
		// if the color is already picked, remove it
		if (color.picked) {
			color.picked = false;
			pickedColors--;
			threadColors = [...threadColors];
			return;
		}

		// if the color cap is reached do nothing
		if (threadColors.filter((c) => c.picked).length >= threadColorCap) {
			return;
		}

		// pick the color
		color.picked = true;
		pickedColors++;
		threadColors = [...threadColors];
	}

	function clickFabricColor(color: Color) {
		// if the color is do nothing
		if (color.picked) {
			return;
		}

		// unselect all other colors
		fabricColors.forEach((c) => (c.picked = false));

		color.picked = true;
		fabricColors = [...fabricColors];
	}

	function generateNewDesigns() {
		// create all possible color variations
		const pickedColors = threadColors.filter((c) => c.picked).map((c) => c.hex);
		let colorVariations: string[][] = [];
		recurseVariations([], totalPaths, pickedColors, colorVariations);

		// generate new designs
		let newResults: Result[] = [];
		const pickedSvg = pickedDesign.svg;

		colorVariations.forEach((variation) => {
			const newSvg = pickedSvg?.cloneNode(true) as HTMLElement;
			const newSvgPaths = newSvg.querySelectorAll('path');

			let colorIndex = 0;
			// set the outline and fill colors of the paths
			for (const path of newSvgPaths) {
				const hasFill = path.style.fill && path.style.fill !== 'none';
				if (hasFill) {
					path.style.fill = variation[colorIndex];
					colorIndex++;
				}

				const hasOutline = path.style.stroke && path.style.stroke !== 'none';
				if (hasOutline) {
					path.style.stroke = variation[colorIndex];
					colorIndex++;
				}
			}
			// set the svg background color
			const pickedFabricColor = fabricColors.find((c) => c.picked);
			if (pickedFabricColor) {
				newSvg.style.backgroundColor = pickedFabricColor.hex;
			}

			newResults.push({ name: variation.join(','), svg: newSvg });
		});

		results = newResults;
	}

	function recurseVariations(
		currentVariation: string[],
		remainingLength: number,
		colors: string[],
		variations: string[][]
	) {
		if (remainingLength === 0) {
			variations.push(currentVariation);
			return;
		}

		for (const color of colors) {
			let extendedVariation = [...currentVariation, color];
			recurseVariations(extendedVariation, remainingLength - 1, colors, variations);
		}
	}
</script>

<h1>Scythe Design Generator</h1>
<p>version: {version}</p>
<p>This will generate {varAmount} variations</p>

<section id="setup">
	<div class="corner-16">
		<div class="card-headline">
			<h2>Thread Colors</h2>
			<p>picked: {pickedColors}/{threadColorCap}</p>
		</div>
		<!-- Thread color picker -->
		<div class="button-grid">
			{#each threadColors as color}
				<button
					on:click={() => clickThreadColor(color)}
					class="color-tile"
					class:picked={color.picked}
					style="background-color: {color.hex};"
					aria-label="color-{color.hex}"
				/>
			{/each}
		</div>
	</div>
	<!-- Fabric color picker -->
	<div class="corner-16">
		<h2>Fabric Color</h2>
		<div class="button-grid">
			{#each fabricColors as color}
				<button
					on:click={() => clickFabricColor(color)}
					class="color-tile"
					class:picked={color.picked}
					style="background-color: {color.hex};"
					aria-label="color-{color.hex}"
				/>
			{/each}
		</div>
	</div>
	<!-- Design picker -->
	<div class="corner-16">
		<div class="card-headline">
			<h2>Design</h2>
			{#if pickedDesign.hoodie}
				<p>only available for a hoodie</p>
			{/if}
			{#if !pickedDesign.hoodie}
				<p>t-shirt or a hoodie</p>
			{/if}
		</div>
		<div class="design-pick">
			{#each designs as design}
				<button
					on:click={() => pickDesign(design)}
					class:active={design.picked}
					aria-label="design-{design.name}">{design.name.toUpperCase()}</button
				>
			{/each}
		</div>
		<!-- design preview -->
		{#if initFlag}
			<div class="corner-8">
				{@html pickedDesign.svg?.outerHTML}
			</div>
		{/if}
	</div>
	<div class="button-tile">
		<button
			on:click={() => generateNewDesigns()}
			id="generate"
			disabled={varAmount === 0}
			aria-label="generate">GENERATE</button
		>
	</div>
</section>
<!-- Results tile -->
<div class="corner-16">
	<h2>Results</h2>
	<div id="results">
		{#each results as result}
			<div class="result-container corner-8">
				<p>{result.name}</p>
				{@html result.svg.outerHTML}
			</div>
		{/each}
	</div>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;700&display=swap');

	:root {
		--col-phlox: #ee00ff;
		--col-canary: #fff300;
		--col-cornsilk: #fff3cf;
		--col-cyan: #00f0e1;
		--col-black: #1d1d1d;

		--col-cornsilk-transp: #fff3cf6c;
	}

	:global(body) {
		background-color: var(--col-black);
		color: var(--col-cornsilk);
		font-size: 16px;
		font-family: 'IBM Plex Mono', monospace;
	}

	#setup {
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 1rem;
		margin-bottom: 1rem;
	}

	.card-headline p {
		margin: 0px;
	}

	.card-headline {
		display: flex;
		justify-content: space-between;
	}

	h2 {
		margin-top: 0px;
	}

	.button-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(50px, 1fr));
		gap: 1rem;
	}

	.color-tile {
		width: 50px;
		height: 50px;
		display: inline-block;

		border: 1px solid var(--col-cornsilk);
	}

	.corner-8 {
		--s: 8px; /* the size on the corner */
		--t: 1px; /* the thickness of the border */
		--g: 8px; /* the gap between the border and element */

		padding: calc(var(--g) + var(--t));
		outline: var(--t) solid var(--col-cornsilk); /* the color here */
		outline-offset: calc(-1 * var(--t));
		-webkit-mask: conic-gradient(at var(--s) var(--s), #0000 75%, #000 0) 0 0 /
				calc(100% - var(--s)) calc(100% - var(--s)),
			linear-gradient(#000 0 0) content-box;
	}

	.corner-16 {
		--s: 16px; /* the size on the corner */
		--t: 1px; /* the thickness of the border */
		--g: 16px; /* the gap between the border and element */

		padding: calc(var(--g) + var(--t));
		outline: var(--t) solid var(--col-cornsilk); /* the color here */
		outline-offset: calc(-1 * var(--t));
		-webkit-mask: conic-gradient(at var(--s) var(--s), #0000 75%, #000 0) 0 0 /
				calc(100% - var(--s)) calc(100% - var(--s)),
			linear-gradient(#000 0 0) content-box;
	}

	button {
		font-family: 'IBM Plex Mono', monospace;
	}

	.picked {
		display: flex;
		justify-content: center;
		align-items: center;

		mask: url('$lib/assets/icons/ic-cross.svg');
		-webkit-mask-image: url('$lib/assets/icons/ic-cross.svg');
	}

	.design-pick {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
		gap: 1rem;

		margin-bottom: 8px;
	}

	.design-pick button {
		background-color: var(--col-black);
		color: var(--col-cornsilk);
		border: 1px solid var(--col-cornsilk);
		padding: 8px;

		font-weight: bold;
	}

	.button-tile {
		display: flex;
		justify-content: center;
		align-items: center;
	}

	#generate {
		padding: 1rem;
		background-color: var(--col-cornsilk);
		color: var(--col-black);
		font-weight: bold;
		font-size: 1.5rem;
		border: none;
		box-sizing: border-box;

		margin-top: 2rem;
		margin-bottom: 2rem;
	}

	#generate:hover,
	#generate:disabled {
		background-color: var(--col-cornsilk-transp);
	}

	.design-pick button:hover {
		background-color: var(--col-cornsilk-transp);
		color: var(--col-black);
	}

	.design-pick button.active {
		background-color: var(--col-cornsilk);
		color: var(--col-black);
	}

	.result-container {
		position: relative;
	}

	.result-container p {
		position: absolute;
		bottom: 0px;
		left: 0px;

		padding: 16px;
		margin: 0px;
		z-index: 10;

		font-size: 8px;
		color: white;
		mix-blend-mode: difference;
	}

	#results {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
		gap: 1rem;
	}

	/* set the amount of columns to 1 on screens narrower than 300px */
	@media screen and (max-width: 600px) {
		#setup {
			grid-template-columns: repeat(1, 1fr);
		}
	}

	/* fit the card headline text nicely on smaller layouts */
	@media screen and (max-width: 850px) {
		.card-headline {
			flex-direction: column;
		}

		.card-headline h2 {
			margin-bottom: 4px;
		}

		.card-headline p {
			margin-bottom: 16px;
		}
	}
</style>
