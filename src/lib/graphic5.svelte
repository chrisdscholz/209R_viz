<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import {sankey, sankeyLinkHorizontal} from 'd3-sankey';

    //variable definition
    let data = [];
    let selectedID = null;
    let selectedName = null;
    let selectedIDTrim = null;
    let selectedTypeTrim = null;
    let selectedDate = null;
    let query = '';

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
            othExp: +d['Other Expense']
        }));

        data = rawData;
        if (rawData.length > 0) {
            selectedID = rawData[3].id;
            selectedName = rawData[3].fiName;
            [selectedTypeTrim, selectedIDTrim] = rawData[3].id.split("_");
            selectedDate = rawData[3].date;
            drawChart(selectedID);
            // drawChart('Credit Union_6');
        }

        // selectedID = rawData[0].id;
        // drawChart(selectedID);

        // console.log(rawData);
    });

    function drawChart(id) {
        // //clear previous visual
        // svg.selectAll('*').remove;

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

        // //define nodes and links
        // const graph = {
        //     nodes: [
        //         { name: 'Interest Revenue' },
        //         { name: 'Non-Interest Revenue' },
        //         { name: 'Non-Interest Expense' },
        //         { name: 'Interest Expense'},
        //         { name: 'Revenue' },
        //         { name: 'Operating Income'},
        //         { name: 'Other Expense'},
        //         { name: 'Net Income' }
        //     ],
        //     links: [
        //         { source: 'Interest Revenue', target: 'Revenue', value: record.netIntRev},
        //         { source: 'Non-Interest Revenue', target: 'Revenue', value: record.nonIntRev },
        //         { source: 'Interest Revenue', target: 'Interest Expense', value: record.intRev - record.netIntRev},
        //         { source: 'Revenue', target: 'Non-Interest Expense', value: record.exp },
        //         { source: 'Revenue', target: 'Operating Income', value: record.netIntRev + record.nonIntRev - record.exp },
        //         { source: 'Operating Income', target: 'Other Expense', value: record.netIntRev + record.nonIntRev - record.exp - record.netInc },
        //         { source: 'Operating Income', target: 'Net Income', value: record.netInc }
        //     ]
        // };

        //define nodes and links

        let nodes = [
            { name: 'Interest Revenue' },
            { name: 'Non-Interest Revenue' },
            { name: 'Operating Expense' },
            { name: 'Interest Expense'},
            { name: 'Gross Revenue' },
            { name: 'Operating Income'},
            { name: 'Other Expense'},
            { name: 'Net Income' }
        ];

        let links = [
            // { source: 'Interest Revenue', target: 'Revenue', value: record.netIntRev},
            // { source: 'Non-Interest Revenue', target: 'Revenue', value: record.nonIntRev },
            // { source: 'Interest Revenue', target: 'Interest Expense', value: record.intRev - record.netIntRev},
            // { source: 'Revenue', target: 'Operating Expense', value: record.exp },
            // { source: 'Revenue', target: 'Operating Income', value: record.netIntRev + record.nonIntRev - record.exp },
            // { source: 'Operating Income', target: 'Other Expense', value: record.netIntRev + record.nonIntRev - record.exp - record.netInc },
            // { source: 'Operating Income', target: 'Net Income', value: record.netInc }

            { source: 'Interest Revenue', target: 'Gross Revenue', value: record.intRev},
            { source: 'Non-Interest Revenue', target: 'Gross Revenue', value: record.nonIntRev },
            { source: 'Gross Revenue', target: 'Interest Expense', value: record.intExp},
            { source: 'Gross Revenue', target: 'Operating Expense', value: record.exp },
            { source: 'Gross Revenue', target: 'Operating Income', value: record.netIntRev + record.nonIntRev - record.exp },
            { source: 'Operating Income', target: 'Other Expense', value: record.othExp },
            { source: 'Operating Income', target: 'Net Income', value: record.netInc }
        ];


        //filter out links with zero values
        links = links.filter(link => link.value > 0);

        // // Map source and target names to node indices
        // graph.links.forEach(link => {
        //     link.source = graph.nodes.findIndex(d => d.name === link.source);
        //     link.target = graph.nodes.findIndex(d => d.name === link.target);
        // });

        //determine nodes to retain based on filtered links
        const connectedNodeNames = new Set ();
        links.forEach(link => {
            connectedNodeNames.add(link.source);
            connectedNodeNames.add(link.target);
        })

        //filter unused nodes
        nodes = nodes.filter(node => connectedNodeNames.has(node.name));

        //map source and targets to nodes
        links.forEach(link => {
            link.source = nodes.findIndex(d => d.name === link.source);
            link.target = nodes.findIndex(d => d.name === link.target);
        })

        const svg = d3.select('svg')
            .attr('width', width)
            .attr('height', height);

        //clear previous visual
        svg.selectAll('*').remove();

        //define sankey generator
        const sankeyGenerator = sankey()
            .nodeWidth(20)
            .nodePadding(20)
            .extent([[0, 0], [width, height]]);
        
        // //define sankey with given nodes/links defined above
        // const { nodes, links } = sankeyGenerator({
        //     nodes: graph.nodes.map(d => ({ ...d })),
        //     links: graph.links.map(d => ({ ...d }))
        // });

        //define sankey with given nodes/links defined above
        const { nodes: sankeyNodes, links: sankeyLinks } = sankeyGenerator({
            nodes: nodes.map(d => ({ ...d })),
            links: links.map(d => ({ ...d }))
        });

        //adjust positioning for midway nodes
        sankeyNodes.forEach((d, i) => {
            // if (d.name === 'Operating Expense' || d.name === 'Interest Expense' || d.name === 'Other Expense') {
            //     // d.x0 = (width / 2) - ((d.x1 - d.x0) / 2);
            //     d.x0 = (width * 0.9);
            //     // d.x1 = d.x0 + (d.x1 - d.x0);
            //     d.x1 = d.x0 + 20;
            // }
            if (d.name === 'Operating Income') {
                // d.x0 = (width / 2) - ((d.x1 - d.x0) / 2);
                d.x0 = (width * 0.65);
                // d.x1 = d.x0 + (d.x1 - d.x0);
                d.x1 = d.x0 + 20;
            }
            if (d.name === 'Revenue') {
                // d.x0 = (width / 2) - ((d.x1 - d.x0) / 2);
                d.x0 = (width * 0.45);
                // d.x1 = d.x0 + (d.x1 - d.x0);
                d.x1 = d.x0 + 20;
            }
        });

        //color scale
        const colorScale = d3.scaleOrdinal()
            .domain(nodes.map(d => d.name))
            .range(d3.schemeObservable10);

        //append gradients to svg
        const defs = svg.append('defs');

        sankeyLinks.forEach((link, i) => {
            const gradientId = `gradient-${i}`;

            const sourceNode = link.source;
            const targetNode = link.target;
            
            const sourceColor = colorScale(sourceNode.name);
            const targetColor = colorScale(targetNode.name);

            const gradient = defs.append('linearGradient')
                .attr('id', gradientId)
                .attr('gradientUnits', 'userSpaceOnUse')
                .attr('x1', link.source.x1)
                .attr('y1', (link.source.y1 + link.source.y0) / 2)
                .attr('x2', link.target.x0)
                .attr('y2', (link.target.y1 + link.target.y0) / 2);

            gradient.append('stop')
                .attr('offset', '0%')
                .attr('stop-color', sourceColor);

            gradient.append('stop')
                .attr('offset', '100%')
                .attr('stop-color', targetColor);

            link.gradientId = gradientId;
    });

        //draw links
        svg.append('g')
            .selectAll('path')
            .data(sankeyLinks)
            .join('path')
            .attr('d', sankeyLinkHorizontal())
            .attr('fill', 'none')
            // .attr('stroke', '#69b3a2')
            // .attr('stroke', d => colorScale(d.source.name))
            .attr('stroke', d => `url(#${d.gradientId})`)
            .attr('stroke-width', d => Math.max(1, d.width))
            .attr('opacity', 0.7);

        //draw nodes
        svg.append('g')
            .selectAll('rect')
            .data(sankeyNodes)
            .join('rect')
            .attr('x', d => d.x0)
            .attr('y', d => d.y0)
            .attr('width', d => d.x1 - d.x0)
            .attr('height', d => d.y1 - d.y0)
            // .attr('fill', '#4682b4')
            .attr('fill', d => colorScale(d.name))
            .attr('stroke', '#000');

        //add node labels
        svg.append('g')
            .selectAll('text')
            .data(sankeyNodes)
            .join('text')
            .attr('x', d => d.x0 - 6)
            .attr('y', d => (d.y1 + d.y0) / 2)
            .attr('dy', '0.35em')
            .attr('text-anchor', 'end')
            .attr('font-size', '12px')
            .attr('fill', '#000')
            .text(d => d.name)
            .filter(d => d.x0 < width / 2)
            .attr('x', d => d.x1 + 6)
            .attr('text-anchor', 'start');
    }



</script>

<style>
    svg {
        border: 1px solid #ccc;
    }
</style>

<div>
    <p>FI Name: {selectedName}</p>
    <p>Type: {selectedTypeTrim}</p>
    <p>Charter/Certificate: {selectedIDTrim}</p>
    <p>Report Date: YTD {selectedDate}</p>
</div>

<svg></svg>

<div>
    <label for="idSelect">Select ID:</label>
    <select id="idSelect" bind:value={selectedID} on:change={() => drawChart(selectedID)}>
        {#each data as { id }}
            <option value={id}>{id}</option>
        {/each}
    </select>
</div>

<!-- <div>
    <input 
        type="search"
        bind:value="{query}"
        aria-label="Search FIs"
        placeholder="FI name/charter/cert..."
    />
</div> -->