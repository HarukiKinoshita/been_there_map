
<script>
  import { geoPath, geoMercator } from "d3-geo";
  import { select } from 'd3-selection';
  import { getContext, onMount } from "svelte";
  import { feature } from "topojson";

  import * as htmlToImage from 'html-to-image';
  import Grid from "svelte-grid-responsive";

  import { get } from 'svelte/store'
  import { stored_visited_list_japan } from './stores'

  import * as geodata from './topojson/japan.json'

  const projection = geoMercator()
                    .scale(1200)
                    .center([149, 37]);
  const path = geoPath().projection(projection);

  $: themeColor = "#ff3e00";

  let f_original = [];
  let f_ccaa = [];
  let f_okinawa = [];
  let okinawapath;
  $: mode = f_ccaa;

  let hovered;
  let visited_list = get(stored_visited_list_japan) ? get(stored_visited_list_japan) : {};
  let count = get(stored_visited_list_japan) ? Object.keys(get(stored_visited_list_japan)).length : 0;

  let mousePosition = { x: 0, y: 0 }; 
  let tooltipTarget = null;

  let node;

  onMount(async () => {
    f_original = feature(geodata, geodata.objects.japan)
    f_ccaa = {
      ...f_original,
      features: f_original.features.sort((a, b) => a.properties.id > b.properties.id)
    }.features;
    f_okinawa = {
      ...f_original,
      features: f_original.features.find(
        el => el.properties.id == 47
      )
    }.features;
    console.log(f_okinawa);
    console.log(f_ccaa);

    select(okinawapath).attr("transform", "translate(100, -430)");

    node = document.getElementById('map_container');
  });

  function addToList(properties) {
    if (visited_list[properties.nam_ja]) {
      delete visited_list[properties.nam_ja]
      // make it reactive
      visited_list = visited_list;
      count--;
    }
    else {
      visited_list[properties.nam_ja] = 1;
      // make it reactive
      visited_list = visited_list;
      count++;
    }
    stored_visited_list_japan.set(visited_list);
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
    padding: 2em;
    background-color: hsl(0, 0%, 95%) !important;
  }
  #svg_box {
    border: 1px solid black;
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
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <svg viewBox="0 0 490 520" preserveAspectRatio="xMidYMid meet" on:click={() => {tooltipTarget = null}} id="svg_box">
        <!-- <svg viewBox="0 0 960 500" preserveAspectRatio="xMidYMid meet"> -->
          <g>
            {#each mode.filter(el => el.properties.id !== 47) as feature, i}
              <!-- svelte-ignore a11y-click-events-have-key-events -->
              <path
                id={feature.properties.nam_ja}
                d={path(feature)}
                class="area"
                fill={visited_list[feature.properties.nam_ja] ? themeColor : "#fff"}
                on:mouseover={() => {hovered = feature, showTooltip(feature.properties.nam_ja)}}
                on:mouseleave={() => {tooltipTarget = null}}
                on:focus={() => {hovered = feature}}
                on:click={() => {addToList(feature.properties)}} 
              />
            {/each}
          </g>
          <path
              bind:this={okinawapath}
              d={path(f_okinawa)}
              class="area"
              fill={visited_list["沖縄県"] ? themeColor : "#fff"}
              on:mouseover={() => {hovered = f_okinawa, showTooltip(f_okinawa.properties.nam_ja)}}
              on:mouseleave={() => {tooltipTarget = null}}
              on:focus={() => {hovered = f_okinawa}}
              on:click={() => {addToList(f_okinawa.properties)}}
            ></path>
        </svg>
      </div>
    </Grid>
    <Grid md={12} lg={4}>
      <fieldset style="text-align: left; background-color: white; margin: 1em;">
        <!-- <legend><strong>Département</strong></legend> -->
        {#each f_ccaa as feature}
        <div id="checkboxes">
          <label style="cursor: pointer font-size: 0.75rem;">
            <input
              type="checkbox"
              id={feature.properties.nam_ja}
              name={feature.properties.nam_ja} 
              value={feature.properties.nam_ja} 
              bind:checked={visited_list[feature.properties.nam_ja]}
              on:click={() => {addToList(feature.properties)}}
            >
            {feature.properties.nam_ja}
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

<!-- Mode Change -->
<!-- <div>
  <p>{ hovered?.properties.nam_ja ?? '' }</p>
  <button on:click={() => {mode = "f_ccaa"}}>Comunidades Autónomas</button>
  <button on:click={() => {mode = "f_pp"}}>Provincias</button>
</div> -->

<div style="position: fixed; right: 24px; bottom: 24px;">
  <button on:click={() => {getImage()}} class="button-orange">
    Export Map
  </button>
</div>