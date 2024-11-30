<script>
    import Graphic5 from '$lib/graphic5.svelte';
    import Graphic6 from '$lib/graphic6.svelte';
    import Graphic7 from '$lib/graphic7.svelte';
    import Writeup from '$lib/writeupProto.svelte';

    let query = '';
    let selectedName;
    let selectedTypeTrim;
    let selectedIDTrim;
    let selectedDate;

    let displayData = {};

    function updateData(data) {
        displayData = data;
    }
</script>

<h3>DSC209R Final Project</h3>
<h3>Chris Scholz</h3>

<div>
    <input 
        type="search"
        bind:value="{query}"
        on:change={updateLabels()}
        aria-label="Search FIs"
        placeholder="FI name/charter/cert..."
    />
</div>

<div id="info">
    <p>FI Name: {displayData.fiName}</p>
    <p>Type: {displayData.type}</p>
    <p>Charter/Certificate: {displayData.idNum}</p>
    <p>Report Date: YTD {displayData.date}</p>
</div>

<!-- <Graphic7 {query} onUpdate={updateData} /> -->

<!-- <Graphic5 {query} onUpdate={updateData}/>

<Graphic6 {query} onUpdate={updateData}/> -->

<div class="graphics-container1">
    <div>
        <p>Accounts</p>
        <Graphic7 {query} onUpdate={updateData} />
    </div>
</div>

<div class="graphics-container2">
    <div>
        <p>Sankey</p>
        <Graphic5 {query} onUpdate={updateData}/>
    </div>
    <div>
        <p>Bar</p>
        <Graphic6 {query} onUpdate={updateData}/>
    </div>
</div>

<!-- <Graphic7 {query} /> -->

<p>Project Github Repo: <a href='https://github.com/chrisdscholz/209R_viz' target="_blank">LINK</a></p>

<Writeup />

<style>
    #info {
        width: 100%;
        max-width: 1620px;
        display: flex;
        flex-direction: row;
        gap: 10px;
        padding: 10px;
        box-sizing: border-box;
    }

    #info p {
        width: 100%;
        padding: 15px;
        margin: 0;
        background-color: #f1f1f1;
        border: 1px solid #ddd;
        border-radius: 5px;
        font-size: 16px;
        color: #333;
        text-align: left;
        font-weight: bold;
    }

    input[type="search"] {
        width: 100%;
        max-width: 400px;
        padding: 10px;
        font-size: 16px;
        border: 1px solid #ccc;
        border-radius: 5px;
        box-sizing: border-box;
    }

    .graphics-container1 {
        width: 100%;
        display: flex;
        flex-direction: row;
        gap: 20px;
        justify-content: left;
        align-items: flex-start;
    }

    .graphics-container2 {
        width: 100%;
        display: flex;
        flex-direction: row;
        gap: 20px;
        justify-content: left;
        align-items: flex-start;
    }

    .graphics-container1 p,
    .graphics-container2 p {
        font-weight: bold;
        text-align: center;
        background-color: #f1f1f1;
        border: 1px solid #ddd;
        border-radius: 5px;
        color: #333;
        padding: 5px;
    }
</style>