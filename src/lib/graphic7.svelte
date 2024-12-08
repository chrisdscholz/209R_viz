<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    //variable definition
    let data = [];
    let filteredData = [];
    let selectedID = null;
    let selectedName = null;
    let selectedIDTrim = null;
    let selectedTypeTrim = null;
    let selectedDate = null;
    export let query = '';
    let svg;

    //load and process data
    onMount(async () => {
        const rawData = await d3.csv('./FIData.csv', d => ({
            id: d['Charter/Cert'],
            fiName: d['Inst Name'],
            intRev: +d['Interest Income'],
            nonIntRev: +d['Non-Interest Income'],
            netInc: +d['Net Income'],
            exp: +d['Non-Interest Expense'],
            netIntRev: +d['Net Interest Income'],
            date: d['Report Date'],
            intExp: +d['Interest Expense'],
            othExp: +d['Other Expense'],
            cashDep: +d['Cash & Deposits'],
            invst: +d['Investments'],
            loanLease: +d['Loans & Leases'],
            assets: +d['Assets'],
            otherAsst: +d['Other Assets'],
            liab: +d['Liabilities'],
            liabEq: +d['Liabilities & Equity'],
            type: d['Charter/Cert'].split("_")[0],
            depAccts: +d['Deposit Accounts']

        }));

        data = rawData;
        filteredData = rawData;
        if (rawData.length > 0) {
            selectedID = rawData[3].id;
            selectedName = rawData[3].fiName;
            [selectedTypeTrim, selectedIDTrim] = rawData[3].id.split("_");
            selectedDate = rawData[3].date;
            drawChart(selectedID);
        }

    });

    $: filteredData = data.filter(d => 
        d.id.toLowerCase().includes(query.toLowerCase()) ||
        d.fiName.toLowerCase().includes(query.toLowerCase())
    );

    // $: if (filteredData.length > 0) {
    //     selectedID = filteredData[0].id;
    //     drawChart(selectedID);
    // }

    $: if (filteredData.length > 0) {
        const maxRecord = filteredData.reduce((max, record) =>
            record.depAccts > max.depAccts ? record : max
        );

        selectedID = maxRecord.id;

        drawChart(selectedID);
    }

    function drawChart() {

        //visual size
        const margin = {top: 20, right: 0, bottom: 30, left: 0};
        const width = 1620 - margin.left - margin.right;
        const height = 115 - margin.top - margin.bottom;

        //group by type for aggregated treemap
        const groupedData = d3.group(data, d => d.type);
        
        const hierarchyD = {
            name: 'Root',
            children: Array.from(groupedData, ([key, values]) => ({
                name: key,
                children: values.map(d => ({
                    name: d.id,
                    value: d.depAccts,
                    type: d.type
                }))
            }))
        };

        //aggregated data
        const aggregatedData = Array.from(
            d3.rollups(
                data,
                group => d3.sum(group, d => d.depAccts),
                d => d.type
            ),
            ([type, total]) => ({ name: type, value: total })
        );

        //selected ID data
        const selectedRecord = data.find(d => d.id === selectedID);

        //aggregated hierarchy
        const hierarchyA = {
            name: 'Root',
            children: aggregatedData
        };

        //aggregated root
        const rootA = d3.hierarchy(hierarchyA)
            .sum(d => d.value || 0)
            .sort((a, b) => b.value - a.value);

        //selected ID root
        const rootD = d3.hierarchy(hierarchyD)
            .sum(d => d.value || 0)
            .sort((a, b) => b.value - a.value);

        //base treemmap
        const treemapLayout = d3.treemap()
            .size([width, height])
            .tile(d3.treemapDice)
            .padding(0);
        
        //treemaps for aggregated and selected ID
        treemapLayout(rootA);
        treemapLayout(rootD);

        //colors
        const typeColorScale = d3.scaleOrdinal()
            .domain(aggregatedData.map(d => d.name))
            .range(d3.schemeObservable10);

        const highlightC = d3.schemeObservable10[6];

        //clear previous visual
        d3.select('#treemap').selectAll('*').remove();

        //create svg
        svg = d3.select('#treemap')
            .append('svg')
            .attr('width', width + margin.right + margin.left)
            .attr('height', height + margin.top + margin.bottom);

        //draw aggregated treemap
        const aggregatedNodes = svg.append('g')
            .selectAll('rect')
            .data(rootA.leaves())
            .join('rect')
            .attr('x', d => d.x0)
            .attr('y', d => d.y0)
            .attr('width', d => d.x1 - d.x0)
            .attr('height', d => d.y1 - d.y0)
            .attr('fill', d => typeColorScale(d.data.name))
            .attr('stroke', 'none');

        //draw selected ID
        const detailedNodes = svg.append('g')
            .selectAll('rect')
            .data(rootD.leaves())
            .join('rect')
            .attr('x', d => d.x0)
            .attr('y', d => d.y0)
            .attr('width', d => d.x1 - d.x0)
            .attr('height', d => d.y1 - d.y0)
            .attr('fill', d => (d.data.name === selectedID ? highlightC : 'none'))
            .attr('stroke', highlightC)
            .attr('stroke-width', 2)
            .attr('opacity', d => (d.data.name === selectedID ? 1 : 0));

        //add legend
        addLegend(svg, typeColorScale, rootA.children.map(d => d.data.name), width, margin);
    }

    //legend function
    function addLegend(svg, colorScale, categories, width, margin) {
        const legend = svg.append('g')
            .attr('class', 'legend')
            .attr('transform', `translate(${margin.left}, ${margin.top + 50})`)

        const boxWidth = 300;
        const boxHeight = 30;
        const boxSpacing = 0;
        
        categories.forEach((category, i) => {
            const xOffset = i * (boxWidth + boxSpacing);

            const legendItem = legend.append('g')
                .attr('transform', `translate(${xOffset}, 5)`);
        
            //color swatch
            legendItem.append('rect')
                .attr('width', boxWidth)
                .attr('height', boxHeight)
                .attr('fill', colorScale(category));

            //label
            legendItem.append('text')
                .attr('x', boxWidth / 2)
                .attr('y', boxHeight / 2)
                .attr('dy', '0.35em')
                .attr('text-anchor', 'middle')
                .attr('font-size', '12px')
                .attr('fill', '#fff')
                .attr('font-weight', 'bold')
                .text(category);
        });
    }

    //redraw when selectedID changes
    $: if (data.length > 0 && selectedID) {
        drawChart();
    }
</script>

<div id="treemap"></div>

<!-- <div>
    <label for="idSelect">Select ID:</label>
    <select id="idSelect" bind:value={selectedID} on:change={() => drawChart(selectedID)}>
        {#each data as { id }}
            <option value={id}>{id}</option>
        {/each}
    </select>
</div> -->

<!-- <div>
    <input 
        type="search"
        bind:value="{query}"
        aria-label="Search FIs"
        placeholder="FI name/charter/cert..."
    />
</div> -->