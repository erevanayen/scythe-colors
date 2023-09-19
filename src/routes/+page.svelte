<script lang="ts">
	import { onMount } from 'svelte';

	import designSwirl from '$lib/assets/pattern-noise.svg?raw';
	import designTest from '$lib/assets/pattern-test.svg?raw';

	type Color = {
		hex: string;
		picked: boolean;
	};

	type Design = {
		name: string;
		svg: HTMLElement | null;
		svgUrl: string;
		picked: boolean;
		allPaths?: SVGPathElement[];
		pathsWithFill?: SVGPathElement[];
		pathsWithOutline?: SVGPathElement[];
	};

	const version = '0.0.1';
	const threadColorCap = 3;

	let threadColors: Color[] = [
		{ hex: '#000000', picked: false },
		{ hex: '#fb00ff', picked: false },
		{ hex: '#00f0e1', picked: false },
		{ hex: '#fff900', picked: false },
		{ hex: '#e5e5e5', picked: false }
	];
	let fabricColors: Color[] = [
		{ hex: '#f1f1f1', picked: true },
		{ hex: '#1d1d1d', picked: false },
		{ hex: '#e2b15f', picked: false }
	];

	let designs: Design[] = [
		{
			name: 'test',
			svg: null,
			svgUrl: designTest,
			picked: true
		},
		{ name: 'swirl', svg: null, svgUrl: designSwirl, picked: false }
	];
	let pickedDesign = designs[0];
	let initFlag = false;

	let pickedColors = 0;
	let totalPaths = 0; // pathsWithFill + pathsWithOutline
	$: varAmount = pickedColors ** totalPaths;

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
		// if the color is already picked, remove it
		if (color.picked) {
			color.picked = false;
			fabricColors = [...fabricColors];
			return;
		}

		// unselect all other colors
		fabricColors.forEach((c) => (c.picked = false));

		color.picked = true;
		fabricColors = [...fabricColors];
	}
</script>

<h1>D Generator</h1>
<p>version: {version}</p>
<p>This will generate {varAmount} variations</p>

<section id="setup">
	<div class="big-tile">
		<div class="card-headline">
			<h2>Thread Colors</h2>
			<p>picked: {pickedColors}/{threadColorCap}</p>
		</div>
		<div class="button-grid">
			{#each threadColors as color}
				<button
					on:click={() => clickThreadColor(color)}
					class="color-tile"
					class:picked={color.picked}
					style="background-color: {color.hex};"
				/>
			{/each}
		</div>
	</div>
	<div class="big-tile">
		<h2>Fabric Color</h2>
		<div class="button-grid">
			{#each fabricColors as color}
				<button
					on:click={() => clickFabricColor(color)}
					class="color-tile"
					class:picked={color.picked}
					style="background-color: {color.hex};"
				/>
			{/each}
		</div>
	</div>

	<div class="big-tile">
		<h2>Design</h2>
		<div class="design-pick">
			{#each designs as design}
				<button on:click={() => pickDesign(design)} class:active={design.picked}
					>{design.name.toUpperCase()}</button
				>
			{/each}
		</div>
		{#if initFlag}
			<div>
				{@html pickedDesign.svg?.outerHTML}
			</div>
		{/if}
	</div>
	<div class="button-tile">
		<button id="generate" disabled={varAmount === 0}>GENERATE</button>
	</div>
</section>
<div class="big-tile" id="results">
	<h2>Results</h2>
</div>

<style>
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

	.big-tile {
		border: 1px solid var(--col-cornsilk);
		padding: 16px;
	}

	.big-tile h2 {
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

	.picked {
		border-radius: 25px;
	}

	.design-pick {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
		gap: 1rem;
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

	/* set the amount of columns to 1 on screens narrower than 300px */
	@media screen and (max-width: 600px) {
		#setup {
			grid-template-columns: repeat(1, 1fr);
		}
	}
</style>
