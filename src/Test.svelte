<script>
  import Fuzzy from "svelte-fuzzy";
  let query = "";
  let options = { keys: ["name", "ICAO", "IATA", "city/town"],  threshold: 0.2 };


  import data from './airportdata/GlobalAirportDatabase.json'


  let formatted = [];
</script>

<input bind:value={query} />

<br />

<button on:click={() => (query = "carol")}>Search "carol"</button>
<button on:click={() => (query = "")}>Clear</button>

<Fuzzy {query} {data} {options} bind:formatted />

{ console.log(formatted) }
{#each formatted as result}
  <div>
    {#each result as item}
      {#each item as { key, matches, text }}
        <span style="margin-right: 10px;">
        {key}: {text}
        </span>
      {/each}
    {/each}
  </div>
{/each}