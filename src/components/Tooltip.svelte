<script>
    export let data;
    import {format} from "d3-format";
    const suffixFormat = (d) => format(".2s")(d).replace("G", "B");

    import {fade} from "svelte/transition";
</script>

{#if data?.country}
    <div transition:fade>
        <h2>{data.country}</h2>
        <h3>Population: {suffixFormat(data.population)}</h3>
    </div>
{/if}

<style>
    div {
        position: absolute;
        bottom: 30px;
        right: 0;
        text-align: right;

    }

    h2 {
        font-size: 1.5rem;
        font-weight: 700;
        color: white;
        margin-bottom: 0.25rem;
    }

    h3 {
        font-size: 1.2rem;
        font-weight: 400;
        color: rgba(255, 255, 255, 0.8);
        margin: 0;
    }
</style>