<script>
	import ProgressBar from "$lib/ProgressBar/index.svelte";
	import SelectionModal from "$lib/SelectionModal/index.svelte";
	import { onMount } from "svelte";

	export let name;
	export let url;
	export let gridIndex;

	let loadTime = (gridIndex < 10 ? gridIndex : 10) * 100 + 1000
	let hideModal = true;
	let fakeLoadingForStylePoints = true;

	const pokemonUrl = url.split("/");
	const pokedexId = pokemonUrl[pokemonUrl.length - 2];
	const imgUrl = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokedexId}.png`;

	onMount(() => {
		setTimeout(() => {
			fakeLoadingForStylePoints = false;
		}, loadTime);
	});
</script>

<div class="card" on:click={() => (hideModal = !hideModal)}>
	<ProgressBar {loadTime} />
	<div hidden={fakeLoadingForStylePoints}>
		<h1>{name}</h1>
		<img src={imgUrl} alt={name} />
	</div>
</div>

{#if !hideModal}
	<SelectionModal
		{pokedexId}
		on:destroyModal={(event) => (hideModal = event.detail.hideModal)}
	/>
{/if}

<style>
	.card {
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
		transition: 0.3s;
		padding: 2px 16px;
		text-align: center;
		min-height: 5rem;
	}

	.card:hover {
		box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 0.2);
	}
</style>
