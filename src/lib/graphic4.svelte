<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import {sankey, sankeyLinkHorizontal} from 'd3-sankey';

    //variable definition
    let data = [];
    let selectedID = null;

    //load and process data
    onMount(async () => {
        const rawData = await d3.csv('./FIData.csv', d => ({
            id: d['Charter/Cert'],
            intRev: +d['Interest Income'],
            nonIntRev: +d['Non-Interest Income'],
            netInc: +d['Net Income'],
            exp: +d['Non-Interest Expense'],
            netIntRev: +d['Net Interest Income']
        }));

        data = rawData;
        if (rawData.length > 0) {
            selectedID = rawData[0].id;
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

        //define nodes and links
        const graph = {
            nodes: [
                { name: 'Interest Revenue' },
                { name: 'Non-Interest Revenue' },
                { name: 'Non-Interest Expense' },
                { name: 'Interest Expense'},
                { name: 'Revenue' },
                { name: 'Operating Income'},
                { name: 'Other Expense'},
                { name: 'Net Income' }
            ],
            links: [
                { source: 'Interest Revenue', target: 'Revenue', value: record.netIntRev},
                { source: 'Non-Interest Revenue', target: 'Revenue', value: record.nonIntRev },
                { source: 'Interest Revenue', target: 'Interest Expense', value: record.intRev - record.netIntRev},
                { source: 'Revenue', target: 'Non-Interest Expense', value: record.exp },
                { source: 'Revenue', target: 'Operating Income', value: record.netIntRev + record.nonIntRev - record.exp },
                { source: 'Operating Income', target: 'Other Expense', value: record.netIntRev + record.nonIntRev - record.exp - record.netInc },
                { source: 'Operating Income', target: 'Net Income', value: record.netInc }
            ]
        };

        // Map source and target names to node indices
        graph.links.forEach(link => {
            link.source = graph.nodes.findIndex(d => d.name === link.source);
            link.target = graph.nodes.findIndex(d => d.name === link.target);
        });

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
        
        //define sankey with given nodes/links defined above
        const { nodes, links } = sankeyGenerator({
            nodes: graph.nodes.map(d => ({ ...d })),
            links: graph.links.map(d => ({ ...d }))
        });

        //draw links
        svg.append('g')
            .selectAll('path')
            .data(links)
            .join('path')
            .attr('d', sankeyLinkHorizontal())
            .attr('fill', 'none')
            .attr('stroke', '#69b3a2')
            .attr('stroke-width', d => Math.max(1, d.width))
            .attr('opacity', 0.7);

        //draw nodes
        svg.append('g')
            .selectAll('rect')
            .data(nodes)
            .join('rect')
            .attr('x', d => d.x0)
            .attr('y', d => d.y0)
            .attr('width', d => d.x1 - d.x0)
            .attr('height', d => d.y1 - d.y0)
            .attr('fill', '#4682b4')
            .attr('stroke', '#000');

        //add node labels
        svg.append('g')
            .selectAll('text')
            .data(nodes)
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
    <label for="idSelect">Select ID:</label>
    <select id="idSelect" bind:value={selectedID} on:change={() => drawChart(selectedID)}>
        {#each data as { id }}
            <option value={id}>{id}</option>
        {/each}
    </select>
</div>

<svg></svg>