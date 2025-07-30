<script>
    export let data;
    export let colorScale;

    // Set the minimum and maximum colors based on the color scale
    const minColor = colorScale.range()[0];
    const maxColor = colorScale.range()[1];

    // Set the minimum and maximum values based on the color scale domain
    const minValue = colorScale.domain()[0];
    const maxValue = colorScale.domain()[1];

    // Format the values at either end of the legend for display
    import {format} from "d3-format";
    const suffixFormat = (d) => format(".2s")(d).replace("G", "B");

    $: percentOfMax = (data?.population / maxValue) * 100;

    import {fade} from "svelte/transition";
</script>

<div class="legend">
    <span class='label'>{minValue}</span>
  <div class="bar" 
      style:background="linear-gradient(to right, {minColor} 0%, {maxColor} 100%)"
  >
  {#if percentOfMax}
    <span class='line' style:left={percentOfMax + "%"}
    transition:fade
    />
  {/if}
  </div>
  <span class='label'>{suffixFormat(maxValue)}</span>
</div>

<style>
    
    .legend {
        display: flex;
        flex-direction: row;
        gap: 6px;
    }

    .bar {
        height: 15px;
        width: 100%;
        position:relative;
    }

    .label {
        font-size: 0.85rem;
        color: white;
        flex-shrink: 0;
        user-select: none;
    }

    .line {
        position: absolute;
        top: 0px;
        width: 2px;
        height: 15px;
        background-color: white;
        transition: left 500ms cubic-bezier(1, 0, 0, 1);
    }
</style>