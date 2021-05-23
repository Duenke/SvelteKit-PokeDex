<script>
	import ProgressBar from "$lib/ProgressBar/index.svelte";
	import SelectionModal from "$lib/SelectionModal/index.svelte"

	export let name;
	export let url;
	let hideModal = true;
	let pokemonDetails = getPokemonDetails();

	async function getPokemonDetails() {
		const response = await fetch(url);
		const details = await response.json();

		if (response.ok) {
			return details;
		} else {
			throw new Error(details);
		}
	}

</script>

{#await pokemonDetails}
	<div class="card">
		<h1>{name}</h1>
		<ProgressBar />
	</div>
{:then pokemon}
	<div class="card" on:click={() => hideModal = !hideModal}>
		<h1>{name}</h1>
		<img src={pokemon.sprites.front_default} alt={name} />
	</div>

	<SelectionModal {hideModal} {pokemon} />	
{:catch error}
	<p class="error">{error.message}</p>
{/await}

<style>
	.error {
		color: red;
		font-weight: bold;
		text-align: center;
	}

	.card {
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
		transition: 0.3s;
		padding: 2px 16px;
		text-align: center;
	}

	.card:hover {
		box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 0.2);
	}
</style>
