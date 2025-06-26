<script lang="ts">
	import Rand, { PRNG } from 'rand-seed'
	import { onMount, untrack } from 'svelte'

	const seedRNG = new Rand()
	const getRandomSeed = () => {
		return `${Math.floor(seedRNG.next() * 2 ** 32)}`
	}

	let canvas: HTMLCanvasElement
	let tileWidth = $state(16)
	let tileHeight = $derived(tileWidth)
	let scale = $state(4)
	let algorithm: PRNG = $state(PRNG.sfc32)
	let seed = $state(getRandomSeed())
	let noiseData = $state(new Uint8Array())
	let bitDepthPower = $state(3)
	let bitDepth = $derived(2 ** bitDepthPower)

	$effect(() => {
		console.log('generating pixel data...')
		const rng = new Rand(seed, algorithm)
		const pixelCount = tileWidth * tileHeight
		noiseData = new Uint8Array(pixelCount)
		untrack(() => {
			for (let i = 0; i < pixelCount; i++) {
				noiseData[i] = Math.floor(rng.next() * 256)
			}
		})
		console.log('regenerated pixel data')
	})

	onMount(() => {
		$effect(() => {
			console.log('rendering...')
			const context = canvas.getContext('2d')
			if (!context) return
			const colorCount = 2 ** bitDepth
			const quant = 256 / colorCount
			for (let tileX = 0; tileX < 4; tileX++) {
				const tileOriginX = tileX * tileWidth
				for (let tileY = 0; tileY < 4; tileY++) {
					const tileOriginY = tileY * tileHeight
					for (let i = 0; i < noiseData.length; i++) {
						const x = tileOriginX + (i % tileWidth)
						const y = tileOriginY + Math.floor(i / tileWidth)
						const value = noiseData[i]
						const colorValue = (Math.floor(value / quant) / (colorCount - 1)) * 255
						context.fillStyle = `rgb(${colorValue}, ${colorValue}, ${colorValue})`
						context.fillRect(x, y, 1, 1)
					}
				}
			}
		})
	})
</script>

<div class="flex flex-col p-4">
	<h1 class="mb-4">Welcome to SvelteKit</h1>
	<div class="flex gap-4">
		<div class="flex flex-col">
			<label class="flex flex-col">
				Size: {tileWidth}
				<input type="range" min="2" max="64" bind:value={tileWidth} />
			</label>
			<label class="flex flex-col">
				Bit depth: {bitDepth}
				<input type="range" min="0" max="3" bind:value={bitDepthPower} />
			</label>
			<div class="flex">
				<label>
					Seed:
					<input type="text" bind:value={seed} />
				</label>
				<button onclick={() => (seed = getRandomSeed())}>Randomize</button>
			</div>
			{#each [PRNG.sfc32, PRNG.mulberry32, PRNG.xoshiro128ss] as algo}
				<label>
					<input type="radio" name="algorithm" value={algo} bind:group={algorithm} />
					{algo}
				</label>
			{/each}
		</div>
		<canvas
			bind:this={canvas}
			width={tileWidth * 4}
			height={tileHeight * 4}
			style:width="{tileWidth * 4 * scale}px"
			style:height="{tileHeight * 4 * scale}px"
		>
		</canvas>
	</div>
</div>
