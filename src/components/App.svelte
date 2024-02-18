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

    // Add zooming
    let svg;
    let g;
    let gScale = 0.7 * 0.85

    $: d3.select(svg).call(zoom);
    $: d3.select(g)
        .attr("transform", "translate(110, 50)")
        
    var zoom = d3.zoom()
        .translateExtent([[0, 0], [width, height]])
        .scaleExtent([1, 2])
        .extent([[0, 0], [width, height]])
        .on("zoom", zoomed);

    function zoomed() {
        let k = d3.zoomTransform(this)['k']
        let x = 110 + d3.zoomTransform(this)['x'] * gScale
        let y = 50 + d3.zoomTransform(this)['y'] * gScale

        d3.select(g)
        .attr("transform", "scale(" + k + ") translate(" + x + "," + y + ")");
    }
</script>

<h1> How has Earthquake Activity Changed Overtime in the United States?</h1>
<h4>
    An interactivate visualization by Cristina De La Torre and Kathleen Nguyen illustrating earthquakes with magnitude no less than three recorded from 1970 to 2014. Data from
    the U.S. Geological Survey was used to create this visualization.
</h4>

<main>
    <label style="margin-left:10px">{slider_label}</label>
    <input type="range" min="1970" max="2014" bind:value={slider_value} style="width: 250px; accent-color: #0088ff"/><br/>
</main>

<!-- <svg bind:this={svg} {width} {height}> -->
<svg bind:this={svg} viewBox="0 0 {width} {height}">
    <g bind:this={g} style="scale: 0.85;">
        <!-- state shapes -->
        <g fill="#e9e9e9" stroke="#737373">
            {#each states as feature, i}
                <path d={path(feature)}  class="state" in:draw={{ delay: 0, duration: 1000 }} />
            {/each}
        </g>

        <!-- points -->
        <g stroke="#1c1cff" stroke-opacity="0.3">
            {#each quakeData as d, i}
                {#if d[3] === slider_value.toString()}
                    <circle
                        key={i}
                        cx={d[0]}
                        cy={d[1]}
                        r={d[2]*1.2}
                        on:mouseover={function(event) {selected_datapoint = d; setMousePosition(event)}}
                        on:mouseout={function() {selected_datapoint = undefined}}
                        class:selected="{selected_datapoint && d.i == d}"
                    />
                {/if}
            {/each}
        </g>
    </g>
</svg>

<style>
    h1{
        display: block;
        text-align: center;
        font-family: "Roboto Serif", serif;
        font-weight: 300;
        margin-bottom: 10px;
    }

    h4{
        display: block;
        text-align: center;
        font-family: "Arial", arial;
        font-weight: 100;
        margin-bottom: 30px;
        margin-left: 200px;
        margin-right: 200px;
        color: gray
    }

    svg {
        display: block;
        margin-top: 0px;
        margin-left: 0px;
        margin-right: 0px;
        margin-bottom: 0px;
        scale: 0.7;
        transform: translate(0px, -150px);
    }

    main{
        font-family: "Arial", arial;
        font-weight: 300;
        text-align: left;
        float: center;
        margin-left: 180px;
        width: 200px;
    }    

    #tooltip {
        font-family: "Arial", arial;
        font-weight: 100;
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
        fill: #0088ff;
        fill-opacity: 0.4;
    }

    circle:hover {
      fill: red;
      fill-opacity: 1;
    }

    h3{
        font-family: "Roboto Serif", serif;
        font-weight: 400;
        text-align: left;
        margin-left: 200px;
        margin-right: 200px;
        transform: translate(0px, 10px);
    }

    p{
        text-indent: 40px;
        font-family: "Arial", arial;
        font-weight: 300;
        text-align: left;
        margin-left: 200px;
        margin-right: 200px;
        line-height:25px
    }
</style>

{#if selected_datapoint != undefined}
    <div id="tooltip" style="left: {mouse_x + 10}px; top: {mouse_y - 10}px">
        Magnitude: {selected_datapoint[2]} <br>
        Longitude: {selected_datapoint[4]} <br>
        Latitude: {selected_datapoint[5]}
    </div>
{/if}

<html lang="en">
    <h3 style="margin-top:-140px">Design Rationale</h3>
    <p>
    The focus of our question lies in analyzing the location of earthquakes across the United States to discover trends in earthquake activity overtime. To explore this question, we chose to plot the location of earthquakes as points using their latitude and longitude across a map of the United States. The points themselves are encoded as blue in order to convey neutrality on the frequency of earthquakes. Although earthquakes are natural disasters, we wanted to encourage the analysis of their patterns rather than emphasize the aspects of danger associated with their frequency. Along with this, the earthquake’s magnitude is encoded as the size of the plotted point. Larger circle markers correspond to earthquakes with greater magnitudes. By doing so, viewers can more closely inspect any variations between earthquake magnitudes across regions. To further explore the data, we chose to construct a slider that displays the number of earthquakes relative to a year. As a result, viewers may explore the amount of seismic activity by year. In this manner, we are able to note the occurrence of drastic changes within a much more narrow scope. We also chose to implement a tool-tip which is used to hover over specific plotted earthquakes on the map, revealing information on the magnitude of the respective earthquake along with its geographical coordinates. We decided to introduce this aspect of interaction because information like the magnitude of the earthquake correlates to the likelihood of successive seismic activity. In this way, the magnitude proves to be useful information in analyzing earthquake occurrences and their frequency. Furthermore, using the tool-tip to hover over a certain point changes the color of the point to red. This color encoding helps distinguish the specific plotted point of interest as the color contrasts the default blue. This provides clarity when using the tool-tip. To further the exploration of the data, the constructed visualization includes another level of interaction where the map can be zoomed into as well as panned. This function was incorporated as it magnifies areas of interest so that particular patterns can be closely analyzed. 
    </p>
    <p style="margin-top:-10px">
    Alternatives considered during the design process included visualizing the data as a line plot of earthquake frequency over the years along with an implementation of a selection feature that expands portions of the plot upon selection of a square of area. Ultimately, we decided to forgo this design concept as it would undermine the complexity of the data. If we were to utilize a line plot, trends explained by locality would not be represented. This is a significant simplification as certain areas experience greater amounts of seismic activity. As such, using a line plot would only be able to provide an overview of patterns for the amount of earthquakes overall in the country, limiting the scope of analysis. We also did not implement the alternative interaction as a horizontal selection feature works more effectively within charts in comparison to its use in a map considering state geometries are not all square. This would result in selections not being smooth, making exploration of the data harder. Another interaction technique considered instead of a slider was a dropdown box listing years as filtering options that would alter the view of the map to only be earthquakes within the specified year. Although the intent of this technique is similar to the slider, we chose to not utilize it as it would impede the ability to smoothly view changes between years. Hence, the slider was chosen as it encodes the continuity of time, allowing the fluidity of seismic changes to be visualized.   
    </p>  
    
    <h3>Development Process</h3>
    <p>
    Together, we began by selecting a dataset that revolved around a universally relevant yet interesting topic and contained a few features that could be explored further. In this manner, we came to use a dataset on detected earthquakes found <a href= "https://github.com/BuzzFeedNews/2015-03-earthquake-maps/tree/master" > here </a> that uses data from the U.S. Geological Survey. After this, we constructed a question to be answered through the developed visualization and discussed its direction along with its focus. After solidifying which aspects of the data the visualization would highlight, the next steps in our process included building an understanding of the languages and framework we would be utilizing for our implementation. To do this, we individually explored the D3 library along with the utility of Svelte. To construct the website, we implemented a pair programming model, taking turns in coding the visualization and providing helpful information. In constructing the website, the map of the country and the points were created first. After doing so, we then introduced interactivity by creating the previously described interaction techniques that would filter and provide more details on the plotted earthquakes. 
    </p>
    <p style="margin-top:-10px">
    The development of this visualization was a process spanning about a week, within which we allotted around 5 hours across a couple of days to work on it. In total this visualization took around 18 hours to fully complete. In reflecting on the development process, a large portion of time was allotted to researching the D3 library and specifying the direction of our visualization along the way. Understanding how components of this visualization worked in conjunction to others played a crucial part in creating the result. In relation to this, the aspect that took the most time was the implementation of the zoom feature. This is due to the complexity of zooming as it we needed to construct so that it behaved in the particular way we wanted it to. These behaviors include the way in which it pans, the smoothness of zooming in, and the extent of how far can the map be zoomed in and out of. Hence, incorporating interactivity was a significant aspect to the final visualization which was reflected in the time taken in its development. 
    </p>
</html>
