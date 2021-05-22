<script>
	$: gridItems = [];
	// $: console.log(gridItems);

	const getApiData = (nextPage) => {
		// gridItems = [];

		fetch(nextPage)
			.then((response) => response.json())
			.then((data) => {
				let results = data.results;

				gridItems = [...gridItems, ...results];

				if (data.next != null) {
					getApiData(data.next);
				}
			});
	};
</script>

<svelte:head>
	<title>PokeDex</title>
</svelte:head>

<button on:click={() => getApiData("https://pokeapi.co/api/v2/pokemon/?limit=1")}
	>Call Pokemon!</button
>

{#each gridItems as item}
	<div class="card">
		<div class="container">
			<h4><b>{item.name}</b></h4>
			<p>{item.url}</p>
		</div>
	</div>
{/each}

<style>
	.card {
		/* Add shadows to create the "card" effect */
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
		transition: 0.3s;
	}

	/* On mouse-over, add a deeper shadow */
	.card:hover {
		box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 0.2);
	}

	/* Add some padding inside the card container */
	.container {
		padding: 2px 16px;
	}
</style>
