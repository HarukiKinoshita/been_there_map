
<script>
  import * as d3_composite from "d3-composite-projections";
  import { geoAlbers, geoNaturalEarth1, geoPath } from "d3-geo";
  import { getContext, onMount } from "svelte";
  import { feature } from "topojson";

  import { get } from 'svelte/store'
  import { stored_visited_list } from './stores'

  // const projection = geoNaturalEarth1().scale(2000).translate([460,1900]);
  const projection = d3_composite.geoConicConformalSpain();
  const path = geoPath().projection(projection);

  $: themeColor = "#ff3e00";

  let f_ccaa = [];
  let f_pp = [];
  $: mode = f_ccaa;

  let hovered;
  let visited_list = get(stored_visited_list) ? get(stored_visited_list) : {};

  let mousePosition = { x: 0, y: 0 }; 
  let tooltipTarget = null;

  onMount(async () => {
    const response = await fetch(
      "https://unpkg.com/es-atlas/es/provinces.json"
    ).then(d => d.json())
    
    // Excluir Ceuta y Melilla
    f_ccaa = feature(response, response.objects.autonomous_regions).features.filter(el => !el.properties.name.includes("Ciudad Autónoma de"));
    f_pp = feature(response, response.objects.provinces).features;
  });

  function addToList(properties) {
    document.getElementById("overSound").currentTime = 0;
		document.getElementById("overSound").play();
    if (visited_list[properties.name]) {
      delete visited_list[properties.name]
      // make it reactive
      visited_list = visited_list;
      // document.getElementById(properties.name).style.fill = "#fff";
    }
    else {
      visited_list[properties.name] = 1;
      // make it reactive
      visited_list = visited_list;
      // document.getElementById(properties.name).style.fill = themeColor;
    }
    stored_visited_list.set(visited_list);
  }

  function showTooltip(name) {
		mousePosition.x = event.clientX;
		mousePosition.y = event.clientY;
		tooltipTarget = name;
	}
</script>

<style>
  .area {
    stroke: #ccc;
    stroke-width: 0.6;
    cursor: pointer;
  }
  .area:focus {
    outline: none;
  }
  #map_container {
    margin: auto auto 3vh auto;
    width: 90vw;
    max-width: 640px;
  }
  .card {
    padding: 3px 5px;
    border-radius: 4px;
    background-color: slategray;
    color: #fff;
    pointer-events: none;
  }
  .headline {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}
  #checkboxes {
    display: inline-block;
    margin: 0.5rem 1.5rem 0 0;
  }
</style>


<p>
  <span style="color: slategray">Ya he visitado</span><br>
  <span class="headline">{ Object.keys(visited_list).length }</span><span style="color: slategray; margin-left: 4px; ">/ { mode.length }</span><br>
</p>
<div id="map_container">
  <svg viewBox="300 0 500 500" preserveAspectRatio="xMidYMid meet">
  <!-- <svg viewBox="0 0 960 500" preserveAspectRatio="xMidYMid meet"> -->
    <g>
      {#each mode as feature, i}
        <path
          id={feature.properties.name}
          d={path(feature)}
          class="area"
          fill={visited_list[feature.properties.name] ? themeColor : "#fff"}
          on:mouseover={() => {hovered = feature, showTooltip(feature.properties.name)}}
          on:mouseleave={() => {tooltipTarget = null}}
          on:focus={() => {hovered = feature}}
          on:click={() => {addToList(feature.properties)}} 
        />
      {/each}
    </g>
  </svg>
</div>

<!-- Tooltip -->
{#if tooltipTarget !== null}
<div class="card" style="position: fixed; left: {mousePosition.x+10}px; top: {mousePosition.y+10}px;">
  {tooltipTarget}
</div>
{/if}

<!-- Mode Change -->
<!-- <div>
  <p>{ hovered?.properties.name ?? '' }</p>
  <button on:click={() => {mode = "f_ccaa"}}>Comunidades Autónomas</button>
  <button on:click={() => {mode = "f_pp"}}>Provincias</button>
</div> -->

<div>
  <fieldset style="text-align: left;">
    <legend><strong>Comunidades Autónomas</strong></legend>
    {#each mode as feature}
    <div id="checkboxes">
      <label style="cursor: pointer;">
        <input
          type="checkbox"
          id={feature.properties.name}
          name={feature.properties.name} 
          value={feature.properties.name} 
          bind:checked={visited_list[feature.properties.name]}
          on:click={() => {addToList(feature.properties)}}
        >
        {feature.properties.name}
      </label>
    </div>
    {/each}
  </fieldset>
</div>

<audio id="overSound" preload="auto">
	<source src="./audio/kettei44.mp3" type="audio/mp3">
</audio>