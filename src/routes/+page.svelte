<script lang="ts">
	type Color = {
		hex: string;
		picked: boolean;
	};

	type Design = {
		name: string;
		svg: string;
		picked: boolean;
	};

	const version = '0.0.1';
	const threadColorCap = 3;
	let pickedColors = 0;

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
			name: 'classic',
			svg: `todo`,
			picked: true
		},
		{ name: 'swirl', svg: 'TODO ', picked: false }
	];
	$: pickedDesign = designs[0];

	$: varAmount = pickedColors; // ** (pathsWithFill.length + pathsWithOutline.length);

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

	function clickDesignButton(design: Design) {
		// unselect all other designs
		designs.forEach((d) => (d.picked = false));

		design.picked = true;
		pickedDesign = design;
		designs = [...designs];
	}
</script>

<h1>SCYTHE Design Generator</h1>
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
				<button on:click={() => clickDesignButton(design)} class:active={design.picked}
					>{design.name.toUpperCase()}</button
				>
			{/each}
		</div>
		<div>
			{pickedDesign.svg}
		</div>
	</div>

	<button id="generate">GENERATE</button>
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

	#generate {
		padding: 1rem;
		background-color: var(--col-cornsilk);
		color: var(--col-black);
		font-weight: bold;
		font-size: 1.5rem;
		border: none;
		box-sizing: border-box;
	}

	#generate:hover {
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
