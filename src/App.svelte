<script>
	import Router, { location, link } from 'svelte-spa-router'
	import Home from './Home.svelte';
	import Europe from './Europe.svelte';
	import Spain from './Spain.svelte';
	import France from './France.svelte';
	import Germany from './Germany.svelte';
	import Italy from './Italy.svelte';
	import UnitedKingdom from './UnitedKingdom.svelte';
	import UnitedStates from './UnitedStates.svelte';

	const routes = {
    '/': Home,
		'/europe': Europe,
		'/spain': Spain,
		'/france': France,
		'/germany': Germany,
		'/italy': Italy,
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
	<!-- <div>
		{#each Object.entries(routes) as [title, paragraph]}
			<a href="./#{title}">{paragraph.name}</a>
		{/each}
	</div> -->
	<a href="/">Home</a>
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

<svelte:head>
	<title>Been There Map</title>
	<html lang="en" />
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta name="description" content="Where have you visited? Create your own Been There map and share it with your friends.">
	<meta property="og:url" content="https://been-there-map.h13a.dev/" />
	<meta property="og:title" content="Been There Map" />
	<meta property="og:type" content="website" />
	<meta property="og:description" content="Where have you visited? Create your own Been There map and share it with your friends." />
	<meta property="og:site_name" content="Been There Map" />
	<meta property="og:image" content="https://been-there-map.h13a.dev/been-there-map-logo.png" />
	<meta name="twitter:card" content="summary" />
	<meta name="twitter:site" content="https://been-there-map.h13a.dev/">
</svelte:head>