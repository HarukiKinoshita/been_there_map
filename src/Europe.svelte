
<script>
  import * as d3_composite from "d3-composite-projections";
  import { geoPath, geoConicConformal } from "d3-geo";
  import { geoBonne, geoModifiedStereographicLee } from "d3-geo-projection";
  import { select } from 'd3-selection';
  import { getContext, onMount } from "svelte";
  import { feature } from "topojson";

  import * as htmlToImage from 'html-to-image';

  import { get } from 'svelte/store'
  import { stored_visited_list_europe } from './stores'

  const projection = geoBonne()
    .rotate([-20.0, 0.0])
    .center([0.0, 52.0])
    .parallel(52)
    .translate([260, 240])
    .scale(640)
    .precision(.1)
  const path = geoPath().projection(projection);

  $: themeColor = "#ff3e00";

  let f_nations = [];
  $: mode = f_nations;

  let hovered;
  let visited_list = get(stored_visited_list_europe) ? get(stored_visited_list_europe) : {};
  let count = get(stored_visited_list_europe) ? Object.keys(get(stored_visited_list_europe)).length : 0;

  let mousePosition = { x: 0, y: 0 }; 
  let tooltipTarget = null;

  let node;

  onMount(async () => {
    const response = await fetch(
      "https://raw.githubusercontent.com/leakyMirror/map-of-europe/27a335110674ae5b01a84d3501b227e661beea2b/TopoJSON/europe.topojson"
      // Source: https://github.com/leakyMirror/map-of-europe
    ).then(d => d.json())
    
    f_nations = feature(response, response.objects.europe).features.filter(el => el.id !== "IL");;
    console.log(f_nations);
  });

  function addToList(properties) {
    document.getElementById("overSound").currentTime = 0;
		document.getElementById("overSound").play();
    if (visited_list[properties.NAME]) {
      delete visited_list[properties.NAME]
      // make it reactive
      visited_list = visited_list;
      count--;
    }
    else {
      visited_list[properties.NAME] = 1;
      // make it reactive
      visited_list = visited_list;
      count++;
    }
    stored_visited_list_europe.set(visited_list);
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
      link.download = `visited-communities-map_${new Date().toLocaleDateString('es-ES')}.jpg`;
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
    margin: 0.5rem 1.5rem 0 0;
  }
  #exported_image {
    width: 80vw;
  }
</style>

<div id="wrapper">
  <p>
    <span style="color: slategray">I have been to</span><br>
    <span class="headline">{ count }</span><span style="color: slategray; margin-left: 4px; ">/ { mode.length }</span><br>
  </p>
  <div id="map_container">
    <svg viewBox="0 0 520 450" preserveAspectRatio="xMidYMid meet" on:click={() => {tooltipTarget = null}}>
    <!-- <svg viewBox="0 0 960 500" preserveAspectRatio="xMidYMid meet"> -->
      <g>
        {#each mode as feature, i}
          <path
            id={feature.properties.NAME}
            d={path(feature)}
            class="area"
            fill={visited_list[feature.properties.NAME] ? themeColor : "#fff"}
            on:mouseover={() => {hovered = feature, showTooltip(feature.properties.NAME)}}
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

<div>
  <fieldset style="text-align: left; background-color: white; margin: 1em;">
    <!-- <legend><strong>Countries</strong></legend> -->
    {#each mode as feature}
    <div id="checkboxes">
      <label style="cursor: pointer;">
        <input
          type="checkbox"
          id={feature.properties.NAME}
          name={feature.properties.NAME} 
          value={feature.properties.NAME} 
          bind:checked={visited_list[feature.properties.NAME]}
          on:click={() => {addToList(feature.properties)}}
        >
        {feature.properties.NAME}
      </label>
    </div>
    {/each}
    <button on:click={() => {stored_visited_list_europe.set(null), visited_list = {}, count = 0}} class="button">Reiniciar</button>
  </fieldset>
</div>

<div style="position: fixed; right: 24px; bottom: 24px;">
  <button on:click={() => {getImage()}} class="button-orange">
    Exportar Mapa
  </button>
</div>

<audio id="overSound" preload="auto">
	<source src="./audio/kettei44.mp3" type="audio/mp3">
</audio>