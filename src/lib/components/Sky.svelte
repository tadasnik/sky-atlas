<script>
	import { LayerCake, Svg, Canvas } from 'layercake';
	import { geoAiry } from 'd3-geo-projection';
	import { select, zoom } from 'd3';
	import SkyCanvas from '$lib/components/SkyCanvas.svelte';
	import SkySvg from '$lib/components/SkySvg.svelte';

	import { getContext } from 'svelte';
	import { geoPath, geoClipRectangle } from 'd3-geo';
	import { scaleLinear } from 'd3-scale';

	export let width;
	export let height;
	export let longitude;
	export let latitude;
	export let mag_th;

	let skycanvas;
	let zoomTransform = { k: 1, x: 0, y: 0 };

	const projection = geoAiry;
	const buff = 10;

	$: zoomX = zoom()
		.scaleExtent([1, 15])
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

	const sphere = { type: 'Sphere' };

	// $: smallDim = $width <= $height ? $width : $height;
	$: projectionDummy = projection().clipAngle(90).scale(1);

	$: b = geoPath(projectionDummy).bounds(sphere);

	$: proj_scale = 0.99 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height);

	$: translator = [width / 2, height / 2];

	$: projectionFn = projection()
		.scale(proj_scale) // * zoomTransform.k)
		.translate(translator) //[translator[0] + zoomTransform.x, translator[1] + zoomTransform.y])
		.reflectX(true)
		.rotate([-longitude, -latitude, 0])
		.clipAngle(90)
		.clipExtent(getScreenPixels(zoomTransform))
		//.postclip(geoClipRectangle(0 - buff, 0 - buff, width + buff, height + buff))
		.precision(0.1);

	function getScreenPixels(zoomTransform) {
		var left = Math.abs(zoomTransform.x) / zoomTransform.k;
		var right = Math.abs(zoomTransform.x - width) / zoomTransform.k;
		var top = Math.abs(zoomTransform.y) / zoomTransform.k;
		var bottom = Math.abs(zoomTransform.y - height) / zoomTransform.k;
		return [
			[left, top],
			[right, bottom]
		];
	}

	$: geoPathFn = geoPath(projectionFn);

	$: isVisible = (d) => {
		const visible = geoPathFn(d);
		return visible ? true : false;
	};
</script>

<div class="sky-map" bind:this={skycanvas}>
	<LayerCake>
		<Canvas>
			<SkyCanvas {projectionFn} {mag_th} {zoomTransform} />
		</Canvas>
		<Svg>
			<SkySvg {projectionFn} {mag_th} {zoomTransform} />
		</Svg>
	</LayerCake>
</div>

<style>
	.sky-map {
		width: 100%;
		height: 100%;
	}
</style>
