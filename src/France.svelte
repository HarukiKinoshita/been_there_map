
<script>
  import * as d3_composite from "d3-composite-projections";
  import { geoPath } from "d3-geo";
  import { select } from 'd3-selection';
  import { getContext, onMount } from "svelte";
  import { feature } from "topojson";

  import * as htmlToImage from 'html-to-image';
  import { toPng } from 'html-to-image';

  import { get } from 'svelte/store'
  import { stored_visited_list_france } from './stores'

  const projection = d3_composite.geoConicConformalFrance().translate([245,260]);
  const path = geoPath().projection(projection);

  $: themeColor = "#ff3e00";

  let f_ccaa = [];
  $: mode = f_ccaa;

  let hovered;
  let visited_list = get(stored_visited_list_france) ? get(stored_visited_list_france) : {};
  let count = get(stored_visited_list_france) ? Object.keys(get(stored_visited_list_france)).length : 0;

  let mousePosition = { x: 0, y: 0 }; 
  let tooltipTarget = null;

  let node;

  onMount(async () => {
    const response = await fetch(
      "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/france/fr-departments.json"
    ).then(d => d.json())

    console.log(response.objects.FRA_adm2)
    
    f_ccaa = feature(response, response.objects.FRA_adm2).features;

    console.log(f_ccaa)

    node = document.getElementById('wrapper');
  });

  function addToList(properties) {
    document.getElementById("overSound").currentTime = 0;
		document.getElementById("overSound").play();
    if (visited_list[properties.NAME_2]) {
      delete visited_list[properties.NAME_2]
      // make it reactive
      visited_list = visited_list;
      count--;
    }
    else {
      visited_list[properties.NAME_2] = 1;
      // make it reactive
      visited_list = visited_list;
      count++;
    }
    stored_visited_list_france.set(visited_list);
  }

  function showTooltip(name) {
		mousePosition.x = event.clientX;
		mousePosition.y = event.clientY;
		tooltipTarget = name;
	}

  function getImage() {
    htmlToImage.toPng(node)
    .then(function (dataUrl) {
      var link = document.createElement('a');
      link.download = `been-there-map_${new Date().toLocaleDateString('es-ES')}`;
      link.href = dataUrl;
      link.click();
    })
    .catch(function (error) {
      console.error('oops, something went wrong!', error);
      alert("Something went wrong. ");
    });
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
  #wrapper {
    padding: 1em;
    background-color: hsl(0, 0%, 95%);
  }
  #map_container {
    margin: auto;
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
    margin: 0.5rem 1rem 0 0;
  }
  #exported_image {
    width: 80vw;
  }
</style>

<div id="wrapper">
  <p>
    <span style="color: slategray">J'ai visité</span><br>
    <span class="headline">{ count }</span><span style="color: slategray; margin-left: 4px; ">/ { mode.length }</span><br>
  </p>
  <div id="map_container">
    <svg viewBox="0 0 490 520" preserveAspectRatio="xMidYMid meet" on:click={() => {tooltipTarget = null}}>
    <!-- <svg viewBox="0 0 960 500" preserveAspectRatio="xMidYMid meet"> -->
      <g>
        {#each mode as feature, i}
          <path
            id={feature.properties.NAME_2}
            d={path(feature)}
            class="area"
            fill={visited_list[feature.properties.NAME_2] ? themeColor : "#fff"}
            on:mouseover={() => {hovered = feature, showTooltip(feature.properties.NAME_2)}}
            on:mouseleave={() => {tooltipTarget = null}}
            on:focus={() => {hovered = feature}}
            on:click={() => {addToList(feature.properties)}} 
          />
        {/each}
      </g>
    </svg>
  </div>
</div>

<!-- Tooltip -->
{#if tooltipTarget !== null}
<div class="card" style="position: fixed; left: {mousePosition.x+10}px; top: {mousePosition.y+10}px;">
  {tooltipTarget}
</div>
{/if}

<!-- Mode Change -->
<!-- <div>
  <p>{ hovered?.properties.NAME_2 ?? '' }</p>
  <button on:click={() => {mode = "f_ccaa"}}>Comunidades Autónomas</button>
  <button on:click={() => {mode = "f_pp"}}>Provincias</button>
</div> -->

<div>
  <fieldset style="text-align: left; background-color: white; margin: 1em;">
    <!-- <legend><strong>Département</strong></legend> -->
    {#each mode as feature}
    <div id="checkboxes">
      <label style="cursor: pointer font-size: 0.75rem;">
        <input
          type="checkbox"
          id={feature.properties.NAME_2}
          name={feature.properties.NAME_2} 
          value={feature.properties.NAME_2} 
          bind:checked={visited_list[feature.properties.NAME_2]}
          on:click={() => {addToList(feature.properties)}}
        >
        {feature.properties.NAME_2}
      </label>
    </div>
    {/each}
  </fieldset>
  <button on:click={() => {stored_visited_list_france.set(null), visited_list = {}, count = 0}} class="button">Réinitialiser</button>
</div>

<div style="position: fixed; right: 24px; bottom: 24px;">
  <button on:click={() => {getImage()}} class="button-orange">
    Exporter Carte
  </button>
</div>

<audio id="overSound" preload="auto">
	<source src="./audio/kettei44.mp3" type="audio/mp3">
</audio>