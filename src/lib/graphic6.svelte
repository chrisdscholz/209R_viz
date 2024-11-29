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
            liabEq: +d['Liabilities & Equity']

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

    $: if (filteredData.length > 0) {
        selectedID = filteredData[0].id;
        drawChart(selectedID);
    }

    function drawChart(id) {

        //visual size
        const margin = {top: 40, right: 30, bottom: 100, left: 80};
        const width = 1000 - margin.left - margin.right;
        const height = 700 - margin.top - margin.bottom;

        //filter selected id
        const record = data.find(d => d.id === id);
        if (!record) return;

        selectedName = record.fiName;
        [selectedTypeTrim, selectedIDTrim] = record.id.split("_");
        selectedDate = record.date;

        let bars = [
            { name: 'Cash & Deposits', value: record.cashDep },
            { name: 'Investments', value: record.invst },
            { name: 'Loans & Leases', value: record.loanLease },
            { name: 'Other Assets', value: record.otherAsst },
            { name: 'Liabilities', value: record.liab}
        ];

        //clear container
        d3.select('#chart').selectAll('*').remove();

        //scales
        const x = d3.scaleBand()
            .domain(bars.map(d => d.name))
            .range([0, width])
            .padding(0.2);
        
        const y = d3.scaleLinear()
            .domain([0, d3.max(bars, d => d.value)])
            .nice()
            .range([height, 0]);

        const color = d3.scaleOrdinal()
            .domain(bars.map(d => d.name))
            .range(d3.schemeObservable10);
        
        //create svg
        svg = d3.select('#chart')
            .append('svg')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        //add x axis
        svg.append('g')
            .attr('transform', `translate(0,${height})`)
            .call(d3.axisBottom(x))
            .selectAll('text')
            .attr('transform', 'rotate(-45)')
            .style('text-anchor', 'end');

        //add y axis
        svg.append('g')
            .call(d3.axisLeft(y));

        //draw bars
        svg.selectAll('.bar')
            .data(bars)
            .join('rect')
            .attr('class', 'bar')
            .attr('x', d => x(d.name))
            .attr('y', d => y(d.value))
            .attr('width', x.bandwidth())
            .attr('height', d => height - y(d.value))
            .attr('fill', d => color(d.name));
        
        //add labels
        svg.selectAll('.label')
            .data(bars)
            .join('text')
            .attr('class', 'label')
            .attr('x', d => x(d.name) + x.bandwidth() / 2)
            .attr('y', d => y(d.value) - 5)
            .attr('text-anchor', 'middle')
            .text(d => d.value.toFixed(2));
    }
</script>

<!-- <div>
    <p>FI Name: {selectedName}</p>
    <p>Type: {selectedTypeTrim}</p>
    <p>Charter/Certificate: {selectedIDTrim}</p>
    <p>Report Date: YTD {selectedDate}</p>
</div> -->

<div id="chart"></div>

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