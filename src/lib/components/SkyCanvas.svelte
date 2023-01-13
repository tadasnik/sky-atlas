<script>
	import { getContext } from 'svelte';
	import { scaleCanvas } from 'layercake';
	import { geoPath, geoGraticule10 } from 'd3-geo';
	import { scaleLinear } from 'd3-scale';
	import milkyway from '$lib/data/milkyway.json';
	import stars from '$lib/data/stars.6.json';
	import constellations from '$lib/data/constellations.lines.json';
	import constellations_borders from '$lib/data/constellations_borders.json';
	import ufgs from '$lib/data/ufg.json';

	const { data, width, height, zGet } = getContext('LayerCake');

	const { ctx } = getContext('canvas');

	/** @type {Function} projection - A D3 projection function. Pass this in as an uncalled function, e.g. `projection={geoAlbersUsa}`. */
	export let projectionFn;
	export let mag_th;
	export let zoomTransform;

	const magnitudeScale = scaleLinear().domain([-1.5, 8]).range([5, 0.5]);
	/** @type {String} [stroke='#ccc'] - The shape's stroke color. */
	export let stroke = '#ccc';

	const lineWidthScale = scaleLinear().domain([1, 15]).range([1, 0.1]);
	/** @type {Number} [strokeWidth=1] - The shape's stroke width. */

	/** @type {String} [fill] - The shape's fill color. By default, the fill will be determined by the z-scale, unless this prop is set. */
	export let fill = undefined;

	/** @type {Array} [features] - A list of GeoJSON features. Use this if you want to draw a subset of the features in `$data` while keeping the zoom on the whole GeoJSON feature set. By default, it plots everything in `$data.features` if left unset. */
	const sphere = { type: 'Sphere' };

	$: geoPathFn = geoPath(projectionFn);

	//$: minute_size =

	$: makeEllipse = (d) => {
		const centroid = projectionFn([d.ra, d.dec]);
		const pole = projectionFn([0, 90]);
		const angle =
			Math.atan2(centroid[1] - pole[1], centroid[0] - pole[0]) - (d.GALDIM_ANGLE * Math.PI) / 180;
		// const start = projectionFn([d.st_end[0][0], d.st_end[0][1]]);
		// const end = projectionFn([d.st_end[1][0], d.st_end[1][1]]);

		//const ra_point = projectionFn([d.ra - d.GALDIM_MINAXIS / 60, d.dec]);
		//const radius = Math.abs(centroid[0] - ra_point[0]) / 2;
		const ellipticity = d.GALDIM_MAJAXIS / d.GALDIM_MINAXIS;
		return { coords: centroid, angle: angle, ellipticity: ellipticity };
	};

	$: isVisible = (d) => {
		const visible = geoPathFn(d);
		//const visible = geoPathFn({ type: 'Point', coordinates: d });
		return visible ? true : false;
	};

	$: visibleStars = stars.features.filter((star) => isVisible(star.geometry));

	$: visibleUfgs = ufgs.filter((ufg) =>
		isVisible({ type: 'Point', coordinates: [ufg.ra, ufg.dec] })
	);

	$: {
		if ($ctx && geoPath) {
			// Set the context here since setting it in `$: geoPath` is a circular reference
			geoPathFn.context($ctx);

			scaleCanvas($ctx, $width, $height);
			$ctx.clearRect(0, 0, $width, $height);
			$ctx.translate(zoomTransform.x, zoomTransform.y);
			$ctx.scale(zoomTransform.k, zoomTransform.k);

			// Draw the Milky Way
			milkyway.features.forEach((feature) => {
				if (feature.id === 'ol1') {
					fill = '#F8F8F8';
				} else if (feature.id === 'ol2') {
					fill = '#F0F0F0';
				} else if (feature.id === 'ol3') {
					fill = '#E0E0E0';
				} else {
					fill = '#D8D8D8';
				}
				$ctx.beginPath();
				geoPathFn(feature);

				$ctx.fillStyle = fill;
				$ctx.fill();
			});

			// Draw graticule
			//$ctx.beginPath();
			//geoPathFn(geoGraticule10());
			//$ctx.lineWidth = strokeWidth;
			//$ctx.strokeStyle = '#D8D8D8';
			//$ctx.stroke();

			// Draw constellation borderst
			$ctx.setLineDash([3, 3]);
			constellations_borders.features.forEach((feature) => {
				$ctx.beginPath();
				geoPathFn(feature);
				$ctx.lineWidth = lineWidthScale(zoomTransform.k);
				$ctx.strokeStyle = stroke;
				$ctx.stroke();
			});

			//reset lines

			$ctx.setLineDash([]);

			// Draw sphere
			$ctx.beginPath();
			geoPathFn(sphere);
			$ctx.lineWidth = lineWidthScale(zoomTransform.k) * 2;
			$ctx.strokeStyle = stroke;
			$ctx.stroke();

			// Draw asterisms
			constellations.features.forEach((feature) => {
				$ctx.beginPath();
				geoPathFn(feature);
				$ctx.lineWidth = lineWidthScale(zoomTransform.k);
				$ctx.strokeStyle = stroke;
				$ctx.stroke();
			});

			// Draw visible stars if magnitude < mag_th
			visibleStars.forEach((d) => {
				if (d.properties.mag < mag_th) {
					$ctx.beginPath();
					let coordinates = projectionFn(d.geometry.coordinates);
					//let coordinates = projectionFn(d.co);
					let radius = magnitudeScale(d.properties.mag);
					//let radius = magnitudeScale(d.mag);
					$ctx.arc(
						coordinates[0],
						coordinates[1],
						radius / (zoomTransform.k * 0.8),
						0,
						2 * Math.PI,
						false
					);
					$ctx.fillStyle = 'black';
					$ctx.fill();
				}
			});

			// Draw some galaxies
			visibleUfgs.forEach((ufg) => {
				let ell = makeEllipse(ufg);
				$ctx.beginPath();
				$ctx.ellipse(ell.coords[0], ell.coords[1], 2, 0.5, ell.angle, 0, 2 * Math.PI);
				$ctx.fillStyle = 'grey';
				$ctx.fill();
			});
		}
	}
</script>
