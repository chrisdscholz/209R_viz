<script>
    import Graphic5 from '$lib/graphic5.svelte';
    import Graphic6 from '$lib/graphic6.svelte';
    import Graphic7 from '$lib/graphic7.svelte';
    import Writeup from '$lib/writeupProto.svelte';

    let query = '';
</script>

<h3>DSC209R Final Project - Prototype</h3>
<h3>Chris Scholz</h3>

<Graphic5 {query} />
<Graphic6 {query} />
<Graphic7 {query} />

<div>
    <input 
        type="search"
        bind:value="{query}"
        aria-label="Search FIs"
        placeholder="FI name/charter/cert..."
    />
</div>

<p>Project Github Repo: <a href='https://github.com/chrisdscholz/209R_viz' target="_blank">LINK</a></p>

<Writeup />