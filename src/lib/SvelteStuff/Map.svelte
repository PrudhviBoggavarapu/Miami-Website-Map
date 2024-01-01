<script lang="ts">
	import type { Map, LayerGroup } from 'leaflet';
	import { onMount, onDestroy } from 'svelte';
	import { Button } from 'flowbite-svelte';

	import {
		cleanData,
		createGoogleMapsURL,
		getCurrentLocation,
		responseData
	} from '../shared/stores';
	import CleanData from './CleanData.svelte';
	let L: typeof import('leaflet');

	type LeafletType = typeof L;

	let map: Map | LayerGroup<any>;
	/**
	 * @type {HTMLDivElement}
	 */
	let mapContainer: HTMLDivElement;

	let lib: typeof import('$lib/wasm-lib/pkg/wasm_lib');

	onMount(async () => {
		// import Wasm

		lib = await import('$lib/wasm-lib/pkg/wasm_lib');
		await lib.default();

		// import Leaflet
		L = await import('leaflet');
		type LeafletType = typeof L;
		await import('leaflet/dist/leaflet.css');

		// Initialize the map
		const defaultLocation: [number, number] = [25.8, -80.25]; // Default location
		map = L.map(mapContainer).setView(defaultLocation, 10.3);
		L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
			attribution: '© OpenStreetMap contributors',
			maxZoom: 18,
			className:
				'filter dark:brightness-60 dark:invert dark:contrast-300 dark:saturate-30 dark:custom-hue-rotate dark:custom-brightness'
		}).addTo(map);

		getCurrentLocation();
		var customIcon = L.icon({
			iconUrl: '/WhereYouAre.png', // Replace with the path to your icon
			iconSize: [50, 50], // Larger size of the icon
			iconAnchor: [25, 50], // Point of the icon which will correspond to marker's location (bottom center of the icon)
			popupAnchor: [0, -41] // Point from which the popup should open relative to the iconAnchor (above the marker)
		});

		navigator.geolocation.getCurrentPosition(
			(item) => {
				L.marker([item.coords.latitude, item.coords.longitude], { icon: customIcon })
					.addTo(map)
					.bindPopup('This is where you are');
			},
			(item) => {}
		);

		cleanData.subscribe((value) => {
			map.eachLayer(function (layer) {
				if (layer instanceof L.Marker) {
					map.removeLayer(layer);
				}
			});
			if (value !== null) {
				value.map((item) => {
					L.marker([item.latitude, item.longitude])
						.addTo(map)
						.bindPopup(
							'<b>' +
								item.name +
								'</b><center><a href=' +
								createGoogleMapsURL(item.address) +
								'>Direction</a></center>'
						);
				});
			}
		});
	});

	onDestroy(() => {
		if (map) {
			map.remove();
		}
	});
</script>

<div bind:this={mapContainer} class="h-screen w-screen"></div>
<CleanData></CleanData>

<style>
	.map-tiles {
		filter: brightness(0.6) invert(1) contrast(3) hue-rotate(200deg) saturate(0.3) brightness(0.7);
	}
</style>