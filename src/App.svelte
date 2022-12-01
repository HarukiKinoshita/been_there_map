<script>
	import Router, { location, link } from 'svelte-spa-router'
	import Spain from './Spain.svelte';
	import France from './France.svelte';
	import Germany from './Germany.svelte';
	import UnitedKingdom from './UnitedKingdom.svelte';
	import Europe from './Europe.svelte';
	import Home from './Home.svelte';
	import { MetaTags } from 'svelte-meta-tags';

	const routes = {
    '/': Home,
    '/spain': Spain,
		'/france': France,
		'/germany': Germany,
		'/uk': UnitedKingdom,
		'/europe': Europe,
    '*': Home
  };
</script>

<header>
	Been There Map
</header>
<main>
  <Router routes={routes}></Router>
</main>
<footer>
	<div class="">
		<a href="/#/europe">Europe</a>
		<a href="/#/spain">Spain</a>
		<a href="/#/france">France</a>
		<a href="/#/germany">Germany</a>
		<a href="/#/uk">United Kingdom</a>
	</div>
	<p>&copy; Haruki Kinoshita</p>
</footer>

<style>
	header {
		background-color: slategray;
		color: white;
		text-align: center;
		padding: 10px;
	}
	main {
		text-align: center;
		margin: 0 auto;
	}
	footer {
		color: slategray;
		text-align: center;
		padding: 5vh 0;
	}
</style>

<MetaTags
  title="Been There Map"
  titleTemplate="%s | Been There Map"
  description="Where have you been? Create your own Been There map. "
  canonical="https://been-there-map.h13a.dev/"
  openGraph={{
    url: 'https://been-there-map.h13a.dev/',
    title: 'Been There Map',
    description: 'Where have you been? Create your own Been There map. ',
    images: [
      {
        url: 'https://been-there-map.h13a.dev/been-there-map-logo.png',
        width: 512,
        height: 512,
        alt: 'Been There Map'
      },
    ],
    site_name: 'Been There Map'
  }}
  twitter={{
    cardType: 'summary',
    title: 'Been There Map',
    description: 'Where have you been? Create your own Been There map. ',
    image: 'https://been-there-map.h13a.dev/been-there-map-logo.png',
    imageAlt: 'Been There Map'
  }}
/>