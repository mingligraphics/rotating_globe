<script>
  import world from "./data/world-110m.json";
  import cable from "./data/cable.json"
  import * as topojson from "topojson-client";
  import Glow from "./components/Glow.svelte";

  let countries = topojson.feature(world, world.objects.countries).features;
  let borders = topojson.mesh(world, world.objects.countries, (a, b) => a !== b);
  let cable_lines = cable.features;

  console.log(countries)

  import { geoOrthographic, geoPath } from "d3-geo";

  let width = 400;
$: height = width;
$: console.log({width})

$: projection = geoOrthographic()
    .scale(width / 2)
    .rotate([$xRotation, $yRotation, 0])
    .translate([width / 2, height / 2]);

$: path = geoPath(projection);

import { timer } from "d3-timer";
import { spring } from "svelte/motion";

let xRotation = spring(0, {stiffness: 0.08, damping: 0.4});
let yRotation = spring(0, {stiffness: 0.17, damping: 0.7});
// spring returns an object. If needs to access the value of the variable itself, need to add a $ sign before. 
let degreePerFrame = 1;
// Speed of $xRotation

const t = timer((elapsed) => {
  if (dragging) return;
  $xRotation += degreePerFrame;
}, 0);

let globe;
//bind globe to the svg, equals to d3.select 

//global.call won't work bc globe was undefined at page load, need to use onMount()

import { onMount } from "svelte";
import { drag } from "d3-drag";
import { select } from "d3-selection";

const DRAG_SENSIBILITY = 1;
let dragging = false;

onMount(()=>{
  const element = select(globe);
  element.call(
    drag().on('drag', (event) => {
    $xRotation = $xRotation + event.dx * DRAG_SENSIBILITY;
    $yRotation = $yRotation - event.dy * DRAG_SENSIBILITY;
    dragging = true;
  })
  // the on below is another props one could access through drag()
   .on("end", () => {
    dragging = false;
   }))
})

</script>

<div class="chart-container" bind:clientWidth={width} class:dragging={dragging}>
  <h1>Planet Earth</h1>
<svg {width} {height} bind:this={globe} >
  <Glow />
  <circle 
  cx={width / 2} 
  cy={height / 2} 
  r={width / 2} 
  fill="#C3ADCF" 
  filter="url('#glow')"
/>
  {#each countries as country}
  <path 
    d={path(country)} 
    fill={"#EFE2BE"} 
    stroke="none"
    />
  {/each}
  <path d={path(borders)} fill="none" stroke="grey"/>
  {#each cable_lines as line}
  <path d={path(line)} fill="none" stroke="yellow"/>
  {/each}
</svg>
</div>

<style>
  .chart-container {
    max-width: 468px;
/* the line below forces the left and right margins to be the same */
    margin: 0 auto; 
  }

  :global(body){
    background:#C6E8EB;
  }

  svg{
    overflow:visible;
  }

  .dragging {
    cursor: grabbing;
  }

  path{
    cursor:pointer;
  }

/* Typically, removing focus outline is a bad idea, but in this case, we've already have the outline of the country */
  path:focus{
    outline: none;
  }

  h1{
    color:black;
    font-family: RetinaMedium, sans-serif;
    font-size: 18px;
  }
</style>
