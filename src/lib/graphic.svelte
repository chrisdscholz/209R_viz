<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let data = [];
    let selectedOffType = "All";
    let offTypes = [];
    
    //define color mapping list
    const colorMap = {
        // "All": "#1f77b4",
        "House": "#ff7f0e",
        "Senate": "#2ca02c",
        //"President": "#d62728"
        "President": "#1f77b4"
    };

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
        const margin = { top: 40, right: 30, bottom: 100, left: 80};
        const width = 1000 - margin.left - margin.right;
        const height = 700 - margin.top - margin.bottom;

        //define color
        const tColor = (offType) => colorMap[offType] || "#ccc";

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

        //define $ billions format
        const formatB = d3.format("$.2s");

        //create stacked layout
        const stack = d3.stack()
            .keys(Object.keys(colorMap))
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
            .attr('fill', d => tColor(d.key))
            .selectAll('rect')
            .data(d => d)
            .join('rect')
            .attr('x', d => x(d.data.year))
            .attr('width', x.bandwidth())
            .attr('y', height)//
            .transition()//
            .duration(500)//
            .delay((d, i) => i * 100)
            // .ease(d3.easeBounce)
            // .ease(d3.easeCubic)//
            // .attr('x', d => x(d.data.year))
            .attr('y', d => y(d[1]))
            .attr('height', d => y(d[0]) - y(d[1]));
            // .attr('width', x.bandwidth());

        //draw axes
        svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', `translate(0, ${height})`)
            .call(d3.axisBottom(x));

        svg.append('g')
            .attr('class', 'y-axis')
            .call(d3.axisLeft(y).tickFormat(d => formatB(d).replace("G", "B")));

        //draw legend container
        const legend = svg.append('g')
            .attr('class', 'legend')
            .attr('transform', `translate(40, 10)`);

        //create legend items
        legend.selectAll('legend-item')
            .data(Object.keys(colorMap))
            .join('g')
            .attr('class', 'legend-item')
            .attr('transform', (d, i) => `translate(0, ${i * 20})`)
            .each(function(d) {
                const legendItem = d3.select(this);

                //color square
                legendItem.append('rect')
                    .attr('x', 0)
                    .attr('y', 0)
                    .attr('width', 15)
                    .attr('height', 15)
                    .attr('fill', tColor(d));
                
                //label
                legendItem.append('text')
                    .attr('x', 20)
                    .attr('y', 12)
                    .text(d)
                    .style('font-size', '12px')
                    .attr('alignment-baseline', 'middle');
            });

        //chart title
        svg.append('text')
            .attr('x', width / 2)
            .attr('y', -margin.top / 2)
            .attr('text-anchor', 'middle')
            .style('font-size', '18px')
            .style('font-weight', 'bold')
            .text('How has federal election campaign spending changed over time?');

        //chart sub title
        svg.append('text')
            .attr('x', width / 2)
            .attr('y', 0)
            .attr('text-anchor', 'middle')
            .style('font-size', '12px')
            .style('font-weight', 'bold')
            .text('Federal Election Commission Reported Spend - Elections Years 2004-2024');

        //x axis title
        svg.append('text')
            .attr('x', width / 2)
            .attr('y', height + margin.bottom - 60)
            .attr('text-anchor', 'middle')
            .attr('font-size', '16px')
            .text('Election Year');

        //y axis title
        svg.append('text')
            .attr('transform', 'rotate(-90)')
            .attr('x', -height / 2)
            .attr('y', -margin.left + 20)
            .attr('text-anchor', 'middle')
            .style('font-size', '16px')
            .text('Spend (in billions)');

        //chart sub notes
        svg.append('text')
            .attr('x', 0)
            .attr('y', height + margin.bottom - 30)
            .attr('text-anchor', 'left')
            .attr('font-size', '12px')
            .text('(1) Only includes election committee spend attributed to a specific candidate');

        svg.append('text')
            .attr('x', 0)
            .attr('y', height + margin.bottom - 10)
            .attr('text-anchor', 'left')
            .attr('font-size', '12px')
            .text('(2) Only includes spend that has been reported to the Federal Elections Commission - very recent election cycles may not be fully reported');
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
    <label for="type-select">Elected Office Type:</label>
    <select id="type-select" bind:value={selectedOffType} on:change={handleOffTypeChange}>
        <option value="All">All</option>
        {#each offTypes as offType}
            <option value={offType}>{offType}</option>
        {/each}
    </select>
</div>

<svg id="chart"></svg>