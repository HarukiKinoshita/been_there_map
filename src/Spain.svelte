
<script>
  import * as d3_composite from "d3-composite-projections";
  import { geoPath } from "d3-geo";
  import { select } from 'd3-selection';
  import { getContext, onMount } from "svelte";
  import { feature } from "topojson";

  import * as htmlToImage from 'html-to-image';
  import Grid from "svelte-grid-responsive";

  import { get } from 'svelte/store'
  import { stored_visited_list_spain } from './stores'

  import * as geodata from './topojson/es_provinces.json'

  const projection = d3_composite.geoConicConformalSpain().translate([150,250]);
  const path = geoPath().projection(projection);

  $: themeColor = "#ff3e00";

  let f_ccaa = [];
  let f_ccaa_original = [];
  let f_pp = [];
  let f_canarias = [];
  let canariapath;
  $: mode = f_ccaa;

  let hovered;
  let visited_list = get(stored_visited_list_spain) ? get(stored_visited_list_spain) : {};
  let count = get(stored_visited_list_spain) ? Object.keys(get(stored_visited_list_spain)).length : 0;

  let mousePosition = { x: 0, y: 0 }; 
  let tooltipTarget = null;

  let node;

  onMount(async () => {    
    f_ccaa_original = feature(geodata, geodata.objects.autonomous_regions);
    console.log(f_ccaa_original);
    f_ccaa = {
      ...f_ccaa_original,
      features: f_ccaa_original.features.filter(
        // Excluir Ceuta y Melilla
        el => !el.properties.name.includes("Ciudad Autónoma de")
      )
    }.features;
    f_canarias = {
      ...f_ccaa_original,
      features: f_ccaa_original.features.find(
        // Excluir Canarias
        el => el.properties.name.includes("Canarias")
      )
    }.features;
    f_pp = feature(geodata, geodata.objects.provinces).features;

    // Canarias
    console.log(canariapath);
    select(canariapath)
      .attr("transform", "translate(200, 20)");

    node = document.getElementById('map_container');
  });

  function addToList(properties) {
    if (visited_list[properties.name]) {
      delete visited_list[properties.name]
      // make it reactive
      visited_list = visited_list;
      count--;
    }
    else {
      visited_list[properties.name] = 1;
      // make it reactive
      visited_list = visited_list;
      count++;
    }
    stored_visited_list_spain.set(visited_list);
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
          <span style="color: slategray">Ya he visitado</span><br>
          <span class="headline">{ count }</span><span style="color: slategray; margin-left: 4px; ">/ { mode.length }</span><br>
        </p>
        <div id="map_container">
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          <svg viewBox="0 0 490 520" preserveAspectRatio="xMidYMid meet" on:click={() => {tooltipTarget = null}}>
          <!-- <svg viewBox="0 0 960 500" preserveAspectRatio="xMidYMid meet"> -->
            <g>
              {#each mode.filter(el => !el.properties.name.includes("Canarias")) as feature, i}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
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
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <path
              bind:this={canariapath}
              d={path(f_canarias)}
              class="area"
              fill={visited_list["Canarias"] ? themeColor : "#fff"}
              on:mouseover={() => {hovered = f_canarias, showTooltip(f_canarias.properties.name)}}
              on:mouseleave={() => {tooltipTarget = null}}
              on:focus={() => {hovered = f_canarias}}
              on:click={() => {addToList(f_canarias.properties)}}
            ></path>
          </svg>
        </div>
      </div>
    </Grid>
    <Grid md={12} lg={4}>
      <fieldset style="text-align: left; background-color: white; margin: 1em;">
        <!-- <legend><strong>Comunidades Autónomas</strong></legend> -->
        {#each mode as feature}
        <div id="checkboxes">
          <label style="cursor: pointer; font-size: 0.75rem;">
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
      <button on:click={() => {stored_visited_list_spain.set(null), visited_list = {}, count = 0}} class="button">Reiniciar</button>
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
  <p>{ hovered?.properties.name ?? '' }</p>
  <button on:click={() => {mode = "f_ccaa"}}>Comunidades Autónomas</button>
  <button on:click={() => {mode = "f_pp"}}>Provincias</button>
</div> -->


<div style="position: fixed; right: 24px; bottom: 24px;">
  <button on:click={() => {getImage()}} class="button-orange">
    Exportar Mapa
  </button>
</div>