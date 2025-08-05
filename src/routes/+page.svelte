<script lang="ts">
	import { onMount } from 'svelte';
	import maplibregl from 'maplibre-gl';

	let map: maplibregl.Map;
	let showParcele = true;

	onMount(() => {
		navigator.geolocation.getCurrentPosition((pos) => {
			const { latitude, longitude } = pos.coords;

			map = new maplibregl.Map({
				container: 'map',
				style: {
					version: 8,
					sources: {
						basemap: {
							type: 'raster',
							tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
							tileSize: 256
						},
						...(showParcele && {
							parcele: {
								type: 'vector',
								tiles: ['http://localhost:8080/data/parcele/{z}/{x}/{y}.pbf'],
								maxzoom: 22
							}
						})
					},
					layers: [
						{
							id: 'osm',
							type: 'raster',
							source: 'basemap'
						},
						...(showParcele
							? [
									{
										id: 'parcele-fill',
										type: 'fill',
										source: 'parcele',
										'source-layer': 'parcele', // ❗️Replace with your actual .mbtiles layer name
										paint: {
											'fill-color': '#088',
											'fill-opacity': 0.4
										}
									}
							  ]
							: [])
					]
				},
				center: [longitude, latitude],
				zoom: 14
			});
		});
	});

	function toggleParcele() {
		showParcele = !showParcele;
		location.reload(); // Simplest way to rebuild the map (for now)
	}
</script>

<style>
	#map {
		width: 100vw;
		height: 100vh;
	}

	.controls {
		position: absolute;
		top: 1rem;
		left: 1rem;
		z-index: 10;
		background: white;
		padding: 0.5rem 1rem;
		border-radius: 0.5rem;
		box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
	}
</style>

<div class="controls">
	<label>
		<input type="checkbox" bind:checked={showParcele} on:change={toggleParcele} />
		Show Parcela Layer
	</label>
</div>

<div id="map"></div>
