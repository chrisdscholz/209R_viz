<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let data = [];
    let selectedOffType = "All";
    let offTypes = [];
    
    //load and process data
    onMount(async () => {
        data = await d3.csv('/FEC_Spend_Data.csv', d => ({
            year: +d.CAND_ELECTION_YR,
            offType: d.CAND_OFFICE,
            spend: +d.TRANSACTION_AMT
        }));

        offTypes = Array.from(new Set(data.map(d => d.offType)));
        drawChart();
    });

    function drawChart() {
        const margin = { top: 20, right: 30, bottom: 40, left: 40};
        const width = 800 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        //dynamic query filter
        const filteredData = selectedOffType === "All" ? data: data.filter(d => d.offType === selectedOffType);

        //create stacked data elements
        const years = Array.from(new Set(filteredData.map(d => d.year)));
        const currentOffTypes = selectedOffType === "All" ? offTypes : [selectedOffType];
       
        //structure data for stacking
        const stackedData = d3.rollups(
            filteredData,
            v => d3.sum(v, d => d.spend),
            d => d.year,
            d => d.offType
        ).map(([year, values]) => {
            const obj = {year};
            values.forEach(([offType, spend]) => {
                obj[offType] = spend;
            });
            return obj;
        })

        //color scale
        const color = d3.scaleOrdinal()
            .domain(currentOffTypes)
            .range(d3.schemeCategory10);

        //define axis scales
        const x = d3.scaleBand()
            .domain(years)
            .range([0, width])
            .padding(0.1);
        
        const y = d3.scaleLinear()
            .domain([0, d3.max(stackedData, d => d3.sum(currentOffTypes, offType => d[offType] || 0))])
            .nice()
            .range([height, 0]);

        //create stacked layout
        const stack = d3.stack()
            .keys(currentOffTypes)
            .value((d, key) => d[key] || 0)

        const layers = stack(stackedData);

        //clear previous chart
        d3.select('#chart').selectAll('*').remove();

        //create SVG
        const svg = d3.select('#chart')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left}, ${margin.top})`);

        //draw layers
        svg.selectAll('.layer')
            .data(layers)
            .join('g')
            .attr('fill', d => color(d.key))
            .selectAll('rect')
            .data(d => d)
            .join('rect')
            .attr('x', d => x(d.data.year))
            .attr('y', d => y(d[1]))
            .attr('height', d => y(d[0]) - y(d[1]))
            .attr('width', x.bandwidth());

        //draw axes
        svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', `translate(0, ${height})`)
            .call(d3.axisBottom(x));

        svg.append('g')
            .attr('class', 'y-axis')
            .call(d3.axisLeft(y));
    }

    //dropdown change
    function handleOffTypeChange(event) {
        selectedOffType = event.target.value;
        drawChart();
    }

    //     const svg = d3.select('#chart')
    //         .attr('width', width + margin.left + margin.right)
    //         .attr('height', height + margin.top + margin.bottom)
    //         .append('g')
    //         .attr('transform', `translate(${margin.left}, ${margin.top})`);
        
    //     //define scale
    //     const x = d3.scaleBand()
    //         .domain(filteredData.map(d => d.year))
    //         .range([0, width])
    //         .padding(0.1);

    //     const y = d3.scaleLinear()
    //         .domain([0, d3.max(filteredData, d => d.spend)])
    //         .nice()
    //         .range([height, 0]);

    //     //draw bars
    //     svg.selectAll('.bar')
    //         .data(filteredData)
    //         .enter().append('rect')
    //         .attr('class', 'bar')
    //         .attr('x', d => x(d.year))
    //         .attr('y', d => y(d.spend))
    //         .attr('width', x.bandwidth())
    //         .attr('height', d => height - y(d.spend))
    //         .attr('fill', 'steelblue');
        
    //     //draw axes
    //     svg.append('g')
    //         .attr('class', 'x-axis')
    //         .attr('transform', `translate(0, ${height})`)
    //         .call(d3.axisBottom(x));

    //     svg.append('g')
    //         .attr('class', 'y-axis')
    //         .call(d3.axisLeft(y));
    // }

    // //dropdown change
    // function handleOffTypeChange(event) {
    //     selectedOffType = event.target.value;
    //     d3.select('#chart').selectAll('*').remove();
    //     drawChart();
    // }
</script>


<style>

</style>


<div>
    <label for="type-select">Select Office Type:</label>
    <select id="type-select" bind:value={selectedOffType} on:change={handleOffTypeChange}>
        <option value="All">All</option>
        {#each offTypes as offType}
            <option value={offType}>{offType}</option>
        {/each}
    </select>
</div>

<svg id="chart"></svg>