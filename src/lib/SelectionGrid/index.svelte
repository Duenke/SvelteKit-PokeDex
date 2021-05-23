<script>
	import GridItem from "$lib/GridItem/index.svelte";
	import { onMount } from "svelte";
	import pokemonLogo from "../../../static/PokemonLogo.png";

	$: gridItems = [];

	onMount(() => {
		fetch("https://pokeapi.co/api/v2/pokemon/?limit=100")
			.then((response) => response.json())
			.then((data) => {
				gridItems = data.results;
			});
	});
</script>

<img src={pokemonLogo} alt="PokeApi" height="500" />

<div>
	{#each gridItems as { name, url } (name)}
		<!-- Making two API calls here, mostly to get the image...may refactor to hard-code the base url + Pokemon Id -->
		<GridItem {name} {url} />
	{/each}
</div>

<style>
	img {
		height: 350px; /* crop the image whitespace vertically */
		object-fit: cover;
	}
	div {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr;
	}
</style>
