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
	import Test from './AirportLocation.svelte';

	const routes = {
    '/': Home,
		'/europe': Europe,
		'/spain': Spain,
		'/france': France,
		'/germany': Germany,
		'/italy': Italy,
		'/uk': UnitedKingdom,
		'/us': UnitedStates,
		'/test': Test,
    '*': Home,
  };
</script>

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