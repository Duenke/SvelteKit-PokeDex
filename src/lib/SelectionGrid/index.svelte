<script>
	import GridItem from "$lib/GridItem/index.svelte";
	import pokemonLogo from "../../../static/PokemonLogo.png";

	$: gridItems = [];

	const getPokeDexData = () => {
		fetch("https://pokeapi.co/api/v2/pokemon/?limit=1000")
			.then((response) => response.json())
			.then((data) => {
				gridItems = data.results;
			});
	};
</script>

<img src={pokemonLogo} alt="PokeApi" height="500" />

<button on:click={() => getPokeDexData()}>Call Pokemon!</button>

<div>
	{#each gridItems as item}
		<GridItem name={item.name} url={item.url} />
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
