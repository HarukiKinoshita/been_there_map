<script>
	import Router, { location, link } from 'svelte-spa-router'
	import Spain from './Spain.svelte';
	import Europe from './Europe.svelte'

	const routes = {
    '/': Spain,
    '/spain': Spain,
		'/europe': Europe,
    '*': Spain
  };
</script>

<header>
	{ $location.substr(1) }
</header>
<main>
  <Router routes={routes}></Router>
</main>
<footer>
	<div class="">
		<a href="/#/europe">Europe</a>
		<a href="/#/spain">Spain</a>
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