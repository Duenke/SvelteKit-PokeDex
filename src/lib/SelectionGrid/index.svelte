<script>
	import GridItem from "$lib/GridItem/index.svelte";

	$: gridItems = [];

	const getPokeDexData = () => {
		fetch("https://pokeapi.co/api/v2/pokemon/?limit=1000")
			.then((response) => response.json())
			.then((data) => {
				gridItems = data.results;
			});
	};
</script>

<button on:click={() => getPokeDexData()}>Call Pokemon!</button>

<div>
	{#each gridItems as item}
		<GridItem name={item.name} url={item.url} />
	{/each}
</div>

<style>
	div {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr;
	}
</style>
