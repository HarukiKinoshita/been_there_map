<script>
  import Grid from "svelte-grid-responsive";
</script>

<div id="wrapper">
  <h1 class="headline">Been There Map</h1>
  <p>Select the map you create and get started!</p>
  <div class="menu">
    <Grid container gutter={36}>
      <Grid xs={12} sm={6} md={4} lg={3}>
        <a href="/#/europe">
          <img src="https://hatscripts.github.io/circle-flags/flags/eu.svg" width="36" alt="Europe">
          <br>
          Europe
        </a>
      </Grid>
      <Grid xs={12} sm={6} md={4} lg={3}>
        <a href="/#/spain">
          <img src="https://hatscripts.github.io/circle-flags/flags/es.svg" width="36" alt="Spain">
          <br>
          Spain
        </a>
      </Grid>
      <Grid xs={12} sm={6} md={4} lg={3}>
        <a href="/#/france">
          <img src="https://hatscripts.github.io/circle-flags/flags/fr.svg" width="36" alt="France">
          <br>
          France
        </a>
      </Grid>
      <Grid xs={12} sm={6} md={4} lg={3}>
        <a href="/#/germany">
          <img src="https://hatscripts.github.io/circle-flags/flags/de.svg" width="36" alt="Germany">
          <br>
          Germany
        </a>
      </Grid>
      <Grid xs={12} sm={6} md={4} lg={3}>
        <a href="/#/uk">
          <img src="https://hatscripts.github.io/circle-flags/flags/uk.svg" width="36" alt="United Kingdom">
          <br>
          United Kingdom
        </a>
      </Grid>
      <Grid xs={12} sm={6} md={4} lg={3}>
        <a href="/#/us">
          <img src="https://hatscripts.github.io/circle-flags/flags/us.svg" width="36" alt="United States">
          <br>
          United States
        </a>
      </Grid>
    </Grid>	
  </div>
  <div id="about">
    <p>
      <strong>Been There Map</strong>
      by Haruki Kinoshita
    </p>
    <a href="https://www.buymeacoffee.com/h13a" target="_blank">
      <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 40px !important;width: 142px !important;" >
    </a>
    <script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="h13a" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>
  </div>
</div>

<style>
  #wrapper {
    padding: 1em;
    background-color: hsl(0, 0%, 95%);
  }
  #about {
    margin-top: 12vh;
  }
  .headline {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}
  .menu {
    background-color: #ffffff;
    padding: 2em;
    border-radius: 20px;
    margin: auto;
    max-width: 840px;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }
  .menu a {
    font-size: 1.5em;
    padding: 1em 0;
    margin: auto;
    white-space: nowrap;
  }
</style>