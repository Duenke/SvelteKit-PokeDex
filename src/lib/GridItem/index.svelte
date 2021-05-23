<script>
	import ProgressBar from "$lib/ProgressBar/index.svelte";
	import SelectionModal from "$lib/SelectionModal/index.svelte";

	export let name;
	export let url;
	let hideModal = true;

	const urlArray = url.split("/");
	const indexLocation = urlArray.length - 2;

	const index = urlArray[indexLocation];

	const imgUrl = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${index}.png`;
</script>

<div on:click={() => (hideModal = !hideModal)}>
	<h1>{name}</h1>
	<img src={imgUrl} alt={name} />
</div>

<!-- <SelectionModal {hideModal} {pokemon} /> -->
<style>
	div {
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
		transition: 0.3s;
		padding: 2px 16px;
		text-align: center;
	}

	div:hover {
		box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 0.2);
	}
</style>
