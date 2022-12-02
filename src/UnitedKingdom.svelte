<script>
  import { geoPath, geoAlbers } from "d3-geo";
  import { select } from 'd3-selection';
  import { getContext, onMount } from "svelte";
  import { feature } from "topojson";

  import * as htmlToImage from 'html-to-image';
  import Grid from "svelte-grid-responsive";

  import { get } from 'svelte/store'
  import { stored_visited_list_uk } from './stores'

  const projection = geoAlbers()
    .rotate([-17.0, 0.0])
    .center([-20, 55.8])
    .translate([260, 240])
    .scale(3100)
    .precision(.1)
  const path = geoPath().projection(projection);

  $: themeColor = "#ff3e00";

  let f_counties = [];
  $: mode = f_counties;

  let hovered;
  let visited_list = get(stored_visited_list_uk) ? get(stored_visited_list_uk) : {};
  let count = get(stored_visited_list_uk) ? Object.keys(get(stored_visited_list_uk)).length : 0;

  let mousePosition = { x: 0, y: 0 }; 
  let tooltipTarget = null;

  let node;

  onMount(async () => {
    const response = await fetch(
      "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/united-kingdom/uk-counties.json"
    ).then(d => d.json())
    
    f_counties = feature(response, response.objects.GBR_adm2).features;

    console.log(response.objects.GBR_adm2.geometries.filter(el => el.NAME_1 !== "England" || el.NAME_1 !== "Scotland"))
    node = document.getElementById('map_container');
  });

  function addToList(properties) {
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
    stored_visited_list_uk.set(visited_list);
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

  function setColor(region) {
    switch (region) {
      case "England":
        return "#dc2626"
      case "Scotland":
        return "#2563eb"
      case "Wales":
        return "#65a30d"
      case "Northern Ireland":
        return "#ca8a04"
      default:
        return themeColor
    }

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
    padding: 2em;
    background-color: hsl(0, 0%, 95%) !important;
  }
  #svg_box {
    max-width: 640px;
    margin: auto;
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
  <Grid container>
    <Grid md={12} lg={8}>
      <div id="map_container">
        <p>
          <span style="color: slategray">I have visited</span><br>
          <span class="headline">{ count }</span><span style="color: slategray; margin-left: 4px; ">/ { mode.length }</span><br>
        </p>
        <svg viewBox="0 0 490 520" preserveAspectRatio="xMidYMid meet" on:click={() => {tooltipTarget = null}} id="svg_box">
          <g>
            {#each mode as feature, i}
              <path
                id={feature.properties.NAME_2}
                d={path(feature)}
                class="area"
                fill={visited_list[feature.properties.NAME_2] ? setColor(feature.properties.NAME_1) : "#fff"}
                on:mouseover={() => {hovered = feature, showTooltip(feature.properties.NAME_2)}}
                on:mouseleave={() => {tooltipTarget = null}}
                on:focus={() => {hovered = feature}}
                on:click={() => {addToList(feature.properties)}} 
              />
            {/each}
          </g>
        </svg>
      </div>
    </Grid>
    <Grid md={12} lg={4}>
      <fieldset style="text-align: left; background-color: white; margin: 1em;">
        <!-- <legend><strong>Countries</strong></legend> -->
        {#each mode as feature}
        <div id="checkboxes">
          <label style="cursor: pointer; font-size: 0.75rem;">
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
    </Grid>
  </Grid>
</div>

<!-- Tooltip -->
{#if tooltipTarget !== null}
<div class="card" style="position: fixed; left: {mousePosition.x+10}px; top: {mousePosition.y+10}px;">
  {tooltipTarget}
</div>
{/if}

<div style="position: fixed; right: 24px; bottom: 24px;">
  <button on:click={() => {getImage()}} class="button-orange">
    Export Map
  </button>
</div>