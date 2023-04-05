<script>
  import { geoPath, geoNaturalEarth1 } from "d3-geo";
  import { zoom } from "d3-zoom";
  import { select } from 'd3-selection';
  import { getContext, onMount } from "svelte";
  import { feature } from "topojson";

  import * as htmlToImage from 'html-to-image';
  import Grid from "svelte-grid-responsive";

  import { get } from 'svelte/store'
  // import { stored_visited_list_europe } from './stores'

  import * as geodata from './topojson/world-atlas.json'
  import airports from './airportdata/GlobalAirportDatabase.json'
  $: data = airports;

  let query = '';
  $: selectedAirport = '';

  import Fuzzy from "svelte-fuzzy";
  let options = { keys: ["name", "ICAO", "IATA", "city/town"],  threshold: 0.2 };
  let formatted = [];

  let routes = [
    {
      origin: "KJFK",
      destination: "EGLL",
    },
    {
      origin: "KJFK",
      destination: ""
    }
  ]

  const projection = geoNaturalEarth1()
    .center([0.0, 52.0])
    .translate([240, 50])
    .scale(100)
    .precision(.1)
  const path = geoPath().projection(projection);

  $: themeColor = "#ff3e00";

  let f_nations = [];

  let hovered;

  let mousePosition = { x: 0, y: 0 }; 
  let tooltipTarget = null;

  let node;

  onMount(async () => {
    f_nations = feature(geodata, geodata.objects.countries).features;

    node = document.getElementById('map_container');
  });


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

  function getAirportData(airport, info) {
    let index = airports.findIndex(el => el.ICAO == airport)
    if (index !== -1) {
      return projection([airports[index]['lng'], airports[index]['lat']])[info]
    }
    else {
      return undefined
    }
  }
</script>

<div id="wrapper">
  <Grid container>
    <Grid md={12} lg={10}>
      <div id="map_container">  
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <svg viewBox="0 0 500 240" preserveAspectRatio="xMidYMid meet" on:click={() => {tooltipTarget = null}} id="svg_box">
        <!-- <svg viewBox="0 0 960 500" preserveAspectRatio="xMidYMid meet"> -->
          <g>
            {#each f_nations as feature, i}
              <path
                id={feature.properties.NAME}
                d={path(feature)}
                class="area"
                fill={'#fff'}
              />
            {/each}
          </g>
          <g>
            <!-- {#each airports as airport, i} -->
            {#if getAirportData(selectedAirport, 0) !== undefined}
              <circle
                cx={getAirportData(selectedAirport, 0)}
                cy={getAirportData(selectedAirport, 1)}
                r={1}
                fill={themeColor}
                on:mouseleave={() => {tooltipTarget = null}}
                on:focus={() => {hovered = feature}}
              />
            {/if}
            <!-- {/each} -->
          </g>
        </svg>
      </div>
    </Grid>
    <Grid md={12} lg={2}>
      <input type="text" placeholder="Airport" bind:value={query}>

      <Fuzzy {query} {data} {options} bind:formatted />

      {#each formatted as item}
        <button style="margin-bottom: 5px; cursor: pointer; display: block;" on:click={() => {selectedAirport = item[1][0].text}}>
          {#each item[0] as line}
            {line.text}
          {/each}
          <!-- {item[1][0].text} -->
        </button>
      {/each}
      Currently selected:{ selectedAirport }

    </Grid>
  </Grid>
</div>

<!-- Tooltip -->
{#if tooltipTarget !== null}
<div class="card" style="position: fixed; left: {mousePosition.x+10}px; top: {mousePosition.y+10}px;">
  <strong>{tooltipTarget.name}</strong><br/>
  {tooltipTarget.IATA} | {tooltipTarget.ICAO}
</div>
{/if}

<div style="position: fixed; right: 24px; bottom: 24px;">
  <button on:click={() => {getImage()}} class="button-orange">
    Export Map
  </button>
</div>

<style>
  .area {
    stroke: #ccc;
    stroke-width: 0.1;
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
    /* max-width: 960px; */
    margin: auto;
    /* border: 1px solid black; */
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
  .tag {
    color: gray;
    padding: 1px 3px;
    margin-right: 4px;
  }
</style>