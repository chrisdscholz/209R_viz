<script>
    import { onMount} from 'svelte';
    let content = '';

    onMount(async () => {
        const response = await fetch('/Project3_Writeup.txt');
        content = await response.text();
    })
</script>

<div style="white-space: pre-wrap; max-width: 1000px; margin: 30px;">
    <p>{content}</p>
</div>

<style>
    /* div {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 1;
        margin: 0;
    } */
</style>