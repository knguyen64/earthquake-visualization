<script>
    import {onMount} from svelte
    import * as d3 from 'd3';
    
    let quakeData = [];

    onMount(async () => {

        const res = await fetch('earthquake_states.csv'); 

        const csv = await res.text();

        quakeData = d3.csvParse(csv, d3.autoType)

        console.log(tempData);

    });
</script>