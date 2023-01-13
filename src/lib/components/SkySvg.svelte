<script>
	import { getContext } from 'svelte';
	import { geoPath, geoStereographic, geoGraticule10, geoCircle } from 'd3-geo';
	import { scaleLinear } from 'd3-scale';
	import ufgs from '$lib/data/big_gal.json';

	export let projectionFn;
	export let zoomTransform;
	export let mag_th;

	const { width, height } = getContext('LayerCake');

	$: geoPathFn = geoPath(projectionFn);

	$: makeEllipse = (d) => {
		const centroid = projectionFn([d.ra, d.dec]);
		const pole = projectionFn([0, 90]);
		const angle =
			(Math.atan2(centroid[1] - pole[1], centroid[0] - pole[0]) * 180) / Math.PI -
			90 -
			d.GALDIM_ANGLE;
		// const start = projectionFn([d.st_end[0][0], d.st_end[0][1]]);
		// const end = projectionFn([d.st_end[1][0], d.st_end[1][1]]);

		//const ra_point = projectionFn([d.ra - d.GALDIM_MINAXIS / 60, d.dec]);
		//const radius = Math.abs(centroid[0] - ra_point[0]) / 2;
		const ellipticity = d.GALDIM_MAJAXIS / d.GALDIM_MINAXIS;
		return { coords: centroid, angle: angle, ellipticity: ellipticity };
	};

	$: isVisible = (d) => {
		const visible = geoPathFn({ type: 'Point', coordinates: [d.ra, d.dec] });
		return visible ? true : false;
	};
</script>

<g class="map-group">
	{#each ufgs as ufg}
		{#if isVisible(ufg)}
			{@const ell = makeEllipse(ufg)}
			<ellipse
				cx={ell.coords[0]}
				cy={ell.coords[1]}
				rx={1}
				ry={1 * ell.ellipticity}
				transform="translate({zoomTransform.x}, {zoomTransform.y})
                   scale({zoomTransform.k}, {zoomTransform.k})
                   rotate({ell.angle} {ell.coords[0]} {ell.coords[1]})"
			/>
		{/if}
	{/each}
</g>

<style>
	.map-group {
		margin: auto;
	}
	path {
		fill: none;
		stroke: lightgray;
	}
	.asterisms {
		fill: none;
		stroke: darkgray;
	}
	ellipse {
		fill: yellow;
		stroke: purple;
		stroke-width: 1;
	}
</style>
