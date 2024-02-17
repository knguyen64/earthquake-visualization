<script>
    import {onMount} from 'svelte';
    import * as d3 from 'd3';
    import * as topojson from 'topojson-client';
	import { geoPath, geoAlbersUsa } from 'd3-geo';
	import { draw } from 'svelte/transition';

    const projection = geoAlbersUsa().scale(1300).translate([487.5, 305])
	
	const path = geoPath().projection(null);
    
    let quakeData = [];
    let states = [];
	let counties = []
	let mesh;
	let selected;
	let width = 1400
	let height = 1000

	const points = [
		{ lat: 38.421115245736, long: -82.44432596047203 },
		{ lat: 38.421115245736, long: -82.44432596047203 }

	].map(p => projection([p.long, p.lat]))

    onMount(async () => {
		const us = await fetch('https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json')
			.then(d => d.json())
		console.log({ us })
		
		states = topojson.feature(us, us.objects.states).features;

        // counties = topojson.feature(us, us.objects.counties).features;
		
		mesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);
		
		$: console.log({ states, mesh })
	})

    //Insert data
    onMount(async () => {
        const res = await fetch('earthquake_states.csv'); 

		const csv = await res.text();

		quakeData = d3.csvParse(csv, d3.autoType);

		console.log(quakeData);
    });

	// quakeData.forEach(d => {
	// 	points.push({lat: d.latitude, long: d.longitude});
	// 	d++;
	// })
	$: locData = quakeData.map(d => [d.longitude, d.latitude, d.mag, d.time.slice(0, 4)])
	$: points2 = locData.map(p => projection([p[0], p[1]])).filter(element => element)
	$: console.log(points2)

</script>

<svg viewBox="0 0 {width} {height}">
	<!-- State shapes -->
	<g fill="white" stroke="black">
		{#each states as feature, i}
			<path d={path(feature)} on:click={() => selected = feature} class="state" in:draw={{ delay: 0, duration: 1000 }} />
		{/each}
	</g>
		
	<!-- Interior lines -->
<!-- 	<path d={path(mesh)} fill="none" stroke="black" /> -->
		
	{#if selected}
		<path d={path(selected)} fill="hsl(0 0% 50% / 20%)" stroke="black" stroke-width={2} />
	{/if}
		
	{#each counties as feature, i}
	  <path d={path(feature)} on:click={() => selected = feature} class="state" stroke="rgb(0 0 0 / 10%)" fill="none" />
	{/each}
	
	{#each points as [cx, cy]}
		<circle {cx} {cy} r={10} fill="black" />
		<circle {cx} {cy} r={8} fill="white" />
		<circle {cx} {cy} r={5} fill="black" />
	{/each}	

	{#each points2 as [cx, cy, mag, year]}
		<circle cx={cx} cy={cy} r={5} fill="black" opacity="0.3"/>
	{/each}	
</svg>

<div class="selectedName">{selected?.properties.name ?? ''}</div>
	
<style>
	.state:hover {
		fill: hsl(0 0% 50% / 20%);
	}
	
	.selectedName {
		text-align: center;
		margin-top: 8px;
		font-size: 1.5rem;
	}
</style>