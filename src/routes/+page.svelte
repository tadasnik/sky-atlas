<script>
	import { LayerCake, Svg, Canvas } from 'layercake';
	import { geoAiry } from 'd3-geo-projection';
	import { DateInput } from 'date-picker-svelte';
	import { select, zoom } from 'd3';
	import Sky from '$lib/components/Sky.svelte';
	import SkyCanvas from '$lib/components/SkyCanvas.svelte';

	let date = new Date();
	let skycanvas;
	let zoomTransform = { k: 1, x: 0, y: 0 };
	let width;
	let height;

	$: latitude = 50;
	$: longitude = 0;
	$: mag_th = 5;

	$: zoomX = zoom()
		.scaleExtent([1, 5])
		.translateExtent([
			[0, 0],
			[width, height]
		])
		.on('zoom', handleZoom);

	function handleZoom(e) {
		zoomTransform.k = e.transform.k;
		zoomTransform.x = e.transform.x;
		zoomTransform.y = e.transform.y;
	}

	$: if (skycanvas) {
		select(skycanvas).call(zoomX);
	}
</script>

<div class="outer-wrapper">
	<div class="controls">
		<DateInput bind:value={date} format="yyyy-MM-dd HH:mm" />
		<label>
			<input type="date" placeholder="YYYY-MM-DD" />
			<input type="time" placeholder="HH-MM" />
		</label>
		<label>
			<input type="range" min="0" max="360" bind:value={longitude} step="0.01" />
			<input type="number" bind:value={longitude} min="0" max="360" />
		</label>

		<label>
			<input type="number" bind:value={latitude} min="-90" max="90" />
		</label>

		<label>
			<input type="number" bind:value={mag_th} min="0" max="14" step="0.1" />
		</label>
	</div>
	<div class="map-container" bind:clientWidth={width} bind:clientHeight={height}>
		<Sky {width} {height} {longitude} {latitude} {mag_th} />
	</div>
</div>

<style>
	/*
    The wrapper div needs to have an explicit width and height in CSS.
    It can also be a flexbox child or CSS grid element.
    The point being it needs dimensions since the <LayerCake> element will
    expand to fill it.
  */
	.outer-wrapper {
		position: relative;
		display: flex;
		flex-direction: column;
		justify-content: center;
		width: 100%;
		height: 100%;
	}
	.controls {
		z-index: 300;
		width: 100%;
		padding: 0 0 1vh 0;
		text-align: center;
	}
	.map-container {
		width: 100%;
		height: 100%;
	}
</style>
