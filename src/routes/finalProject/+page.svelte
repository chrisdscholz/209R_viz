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

<h3>Chris Scholz, DSC209R Final Project: Financial Insitution Research Tool</h3>

<div>
    <input 
        type="search"
        bind:value="{query}"
        aria-label="Search FIs"
        placeholder="Type in any FI name..."
    />
</div>

<div id="info">
    <p>FI Name: {displayData.fiName}</p>
    <p>Type: {displayData.type}</p>
    <p>Charter/Certificate: {displayData.idNum}</p>
    <p>Accounts: {new Intl.NumberFormat().format(displayData.depAccts)}</p>
    <p>Report Date: {displayData.date}</p>
</div>

<!-- <Graphic7 {query} onUpdate={updateData} /> -->

<!-- <Graphic5 {query} onUpdate={updateData}/>

<Graphic6 {query} onUpdate={updateData}/> -->

<div class="graphics-container1">
    <div>
        <p>Number of Accounts Compared to Full Market</p>
        <Graphic7 {query} onUpdate={updateData} />
    </div>
</div>

<div class="graphics-container2">
    <div>
        <p>Income Flow (in millions $)</p>
        <Graphic5 {query} onUpdate={updateData}/>
    </div>
    <div>
        <p>Key Balance Sheet Metrics (in millions $)</p>
        <Graphic6 {query} onUpdate={updateData}/>
    </div>
</div>

<!-- <Graphic7 {query} /> -->

<p>Demo Video: <a href='https://youtu.be/WLsZbcDuMKY?si=mUxHMwkgN_8p7dFA' target="_blank">LINK</a></p>
<p>Project Github Repo: <a href='https://github.com/chrisdscholz/209R_viz' target="_blank">LINK</a></p>

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
        border: 2px solid red;
        border-radius: 5px;
        box-sizing: border-box;
        font-weight: bold;
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