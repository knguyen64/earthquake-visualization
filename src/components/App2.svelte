<script>
    import {onMount} from 'svelte';
    import * as d3 from 'd3';
    import * as topojson from 'topojson-client';
	import { draw } from 'svelte/transition';

    const projection = d3.geoAlbersUsa().scale(1300).translate([487.5, 305])
	const path = d3.geoPath().projection(null);
    
    let allQuakeData = [];
    let states = [];
	let mesh;
	let selected;
	let width = 975
	let height = 610

    // Inserts Data
    onMount(async () => {
		const us = await fetch('https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json').then(d => d.json())
        const res = await fetch('earthquake_states.csv'); 
        const csv = await res.text();
		
		states = topojson.feature(us, us.objects.states).features;
		mesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);
		allQuakeData = d3.csvParse(csv, d3.autoType);
		console.log(quakeData);
	})

    // Cleans Data
    $: quakeData = allQuakeData
        .map(d => [projection([d.longitude, d.latitude]), projection([d.longitude, d.latitude]), d.mag, d.time.slice(0, 4), d.longitude, d.latitude])
        .filter(d => d[0] !== null)
        .map(d => [d[0][0], d[1][1], d[2], d[3], d[4], d[5]])

    // Creates Slider
    let slider_value = 1970;
    let slider_label = "";
    $: slider_label = slider_value;

    // Add tooltip
    export let selected_datapoint = undefined;
    let mouse_x, mouse_y;
    const setMousePosition = function(event) {
        mouse_x = event.clientX;
        mouse_y = event.clientY;
        console.log(mouse_x, mouse_y);
    }

</script>

<h1>Earthquakes in the USA</h1>

<main>
    <label>{slider_label}</label>
    <input type="range" min="1970" max="2014" bind:value={slider_value} style="width: 250px"/><br/>
</main>

<svg viewBox="0 0 {width} {height}">
	<!-- state shapes -->
	<g fill="white" stroke="black">
		{#each states as feature, i}
			<path d={path(feature)}  class="state" in:draw={{ delay: 0, duration: 1000 }} />
		{/each}
	</g>

    <!-- points -->
    <g stroke="#000" stroke-opacity="0.2">
        {#each quakeData as d, i}
            {#if d[3] === slider_value.toString()}
                <circle
                    key={i}
                    cx={d[0]}
                    cy={d[1]}
                    r={d[2]}
                    on:mouseover={function(event) {selected_datapoint = d; setMousePosition(event)}}
                    on:mouseout={function() {selected_datapoint = undefined}}
                    class:selected="{selected_datapoint && d.i == d}"
                />
            {/if}
        {/each}
    </g>
</svg>

<style>
    h1{
        display: block;
        /* font: ; */
        text-align: center;
        margin-bottom: 0px;
    }

    svg {
        display: block;
        margin-top: 0;
        border: 2px solid gray;
        scale: 0.7;
        transform: translate(0px, -130px);
    }

    main{
        text-align: left;
        font-size: large;
        float: center;
        /* margin: 0; */
        width: 200px;
    }    

    #tooltip {
        position: fixed;
        background-color: white;
        padding: 3px;
        border: solid 1px;
    }

    svg.tooltip {
        margin: 0px;
        padding: 0px;
    }

    circle {
        fill: blue;
        fill-opacity: 0.4;
    }

    circle:hover {
      fill: red;
      fill-opacity: 1;
    }
</style>

{#if selected_datapoint != undefined}
    <div id="tooltip" style="left: {mouse_x + 10}px; top: {mouse_y - 10}px">
        Magnitude: {selected_datapoint[2]} <br>
        Longitude: {selected_datapoint[4]} <br>
        Latitude: {selected_datapoint[5]}
    </div>
{/if}