<script>
  import world from "$data/world-110m.json";
  import * as topojson from "topojson-client";

  console.log(world);
  
  // Create countries and borders from the world data
  let countries = topojson.feature(world, world.objects.countries).features;
  let borders = topojson.mesh(world, world.objects.countries, (a, b) => a !== b);

  console.log(countries);
  console.log(borders);

  // Projection function — orthographic projection. Create countries and borders
  import { geoOrthographic, geoPath } from "d3-geo";

  let width = 400;
 $: height = width;

  $: projection = geoOrthographic()
    .scale(width / 2) // How big a projection is
    .rotate([$xRotation, $yRotation, 0]) // How it should rotate on an x, y, z axis
    .translate([width / 2, height / 2]); // Where the projection is centered on the screen

  // Create path generator that will draw our shapes
  $: path = geoPath(projection);

  // Import the Glow component
  import Glow from "./components/Glow.svelte";

  // Import color scale and max population
  import {scaleLinear} from "d3-scale";
  import { max } from "d3-array";

  import data from "$data/data.json";
  // Modify countries array to include population data
  countries.forEach((country) => {
    const metadata = data?.find((d) => d.id === country.id);
    if (metadata) {
      country.population = metadata.population; // Add population to each country
      country.country = metadata.country; // Default to 0 if no data found
    }
  });

  // Create a color scale for the data
  const colorScale = scaleLinear()
    .domain([0, max(data, (d) => d.population)]) // Input
    .range(["#26362e", '#0DCC6C']); // Output

  // Autorotate the globe and spring
  import {timer} from "d3-timer";
  import {spring} from "svelte/motion";
  
  let xRotation = spring(0, { stiffness: 0.08, damping: 0.4 });
  let yRotation = spring(0, { stiffness: 0.1, damping: 0.8 });
  let degreesPerFrame = 0.5;

  // Timer to update xRotation
  const t = timer((elapsed) => {
    if (dragging || tooltipData) return; // If dragging, do not rotate
    $xRotation += degreesPerFrame;
    // yRotation stays at 0 - removed the increment
  }, 0);

  // User interaction 
  let globe;
 //Import onMount from svelte
  import { onMount } from "svelte";
  // Import drag from d3
  import {drag} from "d3-drag";
  import {select} from "d3-selection";

  const dragSensitivity = 1;
  let dragging = false;
  
  onMount (() =>{
    const element = select(globe);
      element.call(
        drag().on("drag", (event) => {
          $xRotation = $xRotation + event.dx * dragSensitivity;
          $yRotation = $yRotation - event.dy * dragSensitivity;
          dragging = true;
        })
        .on("end", () => {
          dragging = false;
        })
      );
  })


// Tooltip
let tooltipData;
import Tooltip from "$components/Tooltip.svelte";

// Import geoCentroid to calculate the center of the hovered country
import {geoCentroid} from "d3-geo";
$: if (tooltipData) {
  const center = geoCentroid(tooltipData);
  $xRotation = -center[0];
  $yRotation = -center[1];
  
}

// Draw the borders when focused
import {draw} from "svelte/transition"

// Import Legend component
import Legend from "$components/Legend.svelte";

</script>


<div class="chart-container" bind:clientWidth={width}>
  <h1 class="title">The world population at a glance</h1>
  <h2>Click on a country to view its population in 2021</h2>
  <svg {width} {height} bind:this={globe} class:dragging>
    <!-- Glow -->
    <Glow />

    <!-- Globe background -->
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
    <circle
      cx={width / 2}
      cy={height / 2}
      r={width / 2}
      fill="#1c1c1c"
      filter="url(#glow)"
      on:click={() => {
        tooltipData = null; // Clear tooltip when clicking on the background
      }}
      on:focus={() => {
        tooltipData = null; // Clear tooltip when focusing on the background
      }}
      tabindex="0"
    />

    <!-- Countries -->
    {#each countries.sort((a, b) => b.population - a.population) as country} 
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
      <path d={path(country)} 
      fill={country.population ? colorScale(country?.population) : 0}
      stroke="none"
      on:click={() => {
        tooltipData = country;
      }}
      on:focus={() => {
        tooltipData = country;
      }}
      tabindex="0"

      />
    {/each}

    <!-- Borders -->
    <path d={path(borders)} 
      fill="none"
      stroke="black"
    />

    <!-- Selected border -->
    {#if tooltipData}
    {#key tooltipData.id}
      <path d={path(tooltipData)} 
        fill="none"
        stroke="white"
        stroke-width="2"
        in:draw
      />
    {/key}
    {/if}
  </svg>

  <!-- Tooltip and Legend -->
  <div style="margin-top: 80px;">
    <Tooltip data={tooltipData} />
  </div>
  <Legend {colorScale} data={tooltipData} />
</div>

<style>
  h1, h2 {
    color: white;
    text-align: center;
  }

  h1 {
    font-size: 1.75rem;
    font-weight: 700;
    line-height: 30px;
    margin-bottom: 0.5rem;
  }

  h2 {
    font-size: 1rem;
    font-weight: 200;
    margin-bottom: 2rem;
  }

  @media (max-width: 468px) {
    h1 {
      font-size: 1.2rem;
    }
    h2 {
      font-size: 0.8rem;
    }
  }
  
  .chart-container {
    max-width: 468px;
    margin: 0 auto;
    outline: none; /* Remove outline for focus */
  }

  :global(body) {
    background: rgb(40, 40, 40);
  }

  svg {
    overflow: visible;
    outline: none; /* Remove outline for focus */
  }

  .dragging {
    cursor: grab;
  }

  path {
    cursor: pointer;
  }

  /* Remove outline when path is focused. Typically a bad idea, but we're already highlighting the path */
  path:focus {
    outline: none;
  }

  circle {
    outline: none;
  }

</style>