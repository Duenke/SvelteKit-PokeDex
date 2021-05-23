<script>
	export let name;
	export let url;

	let itemDetails = getPokemonDetails();

	async function getPokemonDetails() {
		const response = await fetch(url);
		const pokemonDetails = await response.json();

		if (response.ok) {
			return pokemonDetails;
		} else {
			throw new Error(pokemonDetails);
		}
	}
</script>

{#await itemDetails}
	<p>...{name}</p>
{:then pokemon}
	<div>
		<h1>{name}</h1>
		<img src={pokemon.sprites.front_default} alt={name} />
	</div>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}

<style>
    p {
        text-align: center;
        font-weight: bold;
    }

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
