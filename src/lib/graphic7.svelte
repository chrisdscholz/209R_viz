<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    //variable definition
    let data = [];
    let selectedID = null;
    let selectedName = null;
    let selectedIDTrim = null;
    let selectedTypeTrim = null;
    let selectedDate = null;
    let query = '';
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
            type: d['Charter/Cert'].split("_")[0]

        }));

        data = rawData;
        if (rawData.length > 0) {
            selectedID = rawData[3].id;
            selectedName = rawData[3].fiName;
            [selectedTypeTrim, selectedIDTrim] = rawData[3].id.split("_");
            selectedDate = rawData[3].date;
            drawChart(selectedID);
        }

        console.log(data.map(d => d.type));
    });

    function drawChart() {

        //visual size
        const margin = {top: 40, right: 30, bottom: 100, left: 80};
        const width = 1000 - margin.left - margin.right;
        const height = 200 - margin.top - margin.bottom;

        //group by type
        const groupedData = d3.group(data, d => d.type);
        const hierarchy = {
            name: 'Root',
            children: Array.from(groupedData, ([key, values]) => ({
                name: key,
                children: values.map(d => ({
                    name: d.id,
                    value: d.assets,
                    type: d.type
                }))
            }))
        };

        const root = d3.hierarchy(hierarchy)
            .sum(d => d.value || 0)
            .sort((a, b) => b.value - a.value);

        const treemapLayout = d3.treemap()
            .size([width, height])
            .tile(d3.treemapDice)
            .padding(0);
        
        treemapLayout(root);

        //color scale
        const typeColorScale = d3.scaleOrdinal()
            .domain(Array.from(groupedData.keys()))
            .range(d3.schemeObservable10);

        //clear previous visual
        d3.select('#treemap').selectAll('*').remove();

        //create svg
        svg = d3.select('#treemap')
            .append('svg')
            .attr('width', width)
            .attr('height', height);

        //draw rectangles
        const nodes = svg.selectAll('g')
            .data(root.leaves())
            .join('g')
            .attr('transform', d => `translate(${d.x0},${d.y0})`);
        
        nodes.append('rect')
            .attr('width', d => d.x1 - d.x0)
            .attr('height', d => d.y1 - d.y0)
            .attr('fill', d =>
                d.data.name === selectedID
                    ? 'green'
                    : typeColorScale(d.data.type))
            .attr('stroke', 'none');

        // //add labels
        // nodes.append('text')
        //     .attr('x', 5)
        //     .attr('y', 15)
        //     .text(d => d.data.name)
        //     .style('font-size', '10px')
        //     .style('fill', 'black');
    }

    //redraw when selectedID changes
    $: if (data.length > 0 && selectedID) {
        drawChart();
    }
</script>

<div id="treemap"></div>

<div>
    <label for="idSelect">Select ID:</label>
    <select id="idSelect" bind:value={selectedID} on:change={() => drawChart(selectedID)}>
        {#each data as { id }}
            <option value={id}>{id}</option>
        {/each}
    </select>
</div>