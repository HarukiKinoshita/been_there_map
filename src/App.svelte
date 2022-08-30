<script>
	import Map from './Map.svelte';
</script>

<header>
	Visited Communities Map
</header>
<main>
	<Map />
</main>
<footer>
	&copy; Haruki Kinoshita
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
		margin: 0;
		padding: 1em;
		margin: 0 auto;
	}
	footer {
		color: slategray;
		text-align: center;
		padding: 5vh 0;
	}
</style>