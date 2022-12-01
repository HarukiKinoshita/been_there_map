<script>
	import Router, { location, link } from 'svelte-spa-router'
	import Home from './Home.svelte';
	import Europe from './Europe.svelte';
	import Spain from './Spain.svelte';
	import France from './France.svelte';
	import Germany from './Germany.svelte';
	import UnitedKingdom from './UnitedKingdom.svelte';
	import UnitedStates from './UnitedStates.svelte';
	import { MetaTags } from 'svelte-meta-tags';

	const routes = {
    '/': Home,
		'/europe': Europe,
		'/spain': Spain,
		'/france': France,
		'/germany': Germany,
		'/uk': UnitedKingdom,
		'/us': UnitedStates,
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
	<div>
		{#each Object.entries(routes) as [title, paragraph]}
			<a href="./#{title}">{paragraph.name}</a>
		{/each}
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
	footer a {
		padding-right: 10px;
		display: inline-block;
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