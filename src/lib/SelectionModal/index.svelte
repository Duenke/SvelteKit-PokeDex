<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";
	import DataBar from "./DataBar.svelte";
	import PokemonSprites from "./PokemonSprites.svelte";

	export let pokedexId;

	let pokemonDetails = getPokemonDetails(pokedexId);

	const femaleColor = "pink";
	const maleColor = "blue";
	const TYPE_COLORS = {
		bug: "green",
		dark: "orange",
		dragon: "purple",
		electric: "orange",
		fairy: "pink",
		fighting: "orange",
		fire: "red",
		flying: "purple",
		ghost: "purple",
		grass: "green",
		ground: "yellow",
		ice: "teal",
		normal: "orange",
		poison: "purple",
		psychic: "red",
		rock: "yellow",
		steel: "gray",
		water: "blue"
	};

	function toTitleCase(string, splitChar) {
		return string
			.toLowerCase()
			.split(splitChar)
			.map((s) => s.charAt(0).toUpperCase() + s.substring(1))
			.join(" ");
	}

	function getFlavorText(flavors) {
		// RANDOM FLAVOR TEXT
		let tempDescription = [];
		tempDescription = flavors
			.filter((flavor) => flavor.language.name === "en")
			.map((item) => item.flavor_text);

		const num = Math.floor(Math.random() * tempDescription.length);

		return tempDescription[num];
	}

	async function getPokemonDetails(pokedexId) {
		const pokemonURL = `https://pokeapi.co/api/v2/pokemon/${pokedexId}/`;
		const pokemonSpeciesURL = `https://pokeapi.co/api/v2/pokemon-species/${pokedexId}/`;

		try {
			const pokemonGeneralResponse = await fetch(pokemonURL);
			const pokemonGeneral = await pokemonGeneralResponse.json();
			if (!pokemonGeneralResponse.ok) {
				throw new Error(pokemonGeneral);
			}

			const pokemonSpeciesResponse = await fetch(pokemonSpeciesURL);
			const pokemonSpecies = await pokemonSpeciesResponse.json();
			if (!pokemonSpeciesResponse.ok) {
				throw new Error(pokemonSpecies);
			}

			const { name, types, sprites, stats, abilities, height, weight } = pokemonGeneral;

			const {
				flavor_text_entries,
				capture_rate,
				growth_rate,
				gender_rate,
				egg_groups,
				evolves_from_species,
				genera
			} = pokemonSpecies;

			// POKEMON THEME (use the last type in the types array to determine)
			const pokemonTheme = TYPE_COLORS[types[types.length - 1].type.name];

			// GENDER RATIO
			const genderRate = gender_rate;
			const genderRatio = {
				female: 12.5 * genderRate,
				male: 12.5 * (8 - genderRate)
			};

			//EGG GROUPS
			const eggGroups = egg_groups.map((group) => toTitleCase(group.name, " ")).join(", ");

			// ABILITIES
			const formattedAbilities = abilities
				.map((ability) => toTitleCase(ability.ability.name, "-"))
				.join(", ");

			// Effort Values
			const evs = stats
				// filter out stats where effort is 0
				.filter((stat) => {
					if (stat.effort > 0) {
						return true;
					}
					return false;
				})
				.map((stat) => `${stat.effort} ${toTitleCase(stat.stat.name, "-")}`)
				.join(", ");

			// EVOLUTION
			let evolvesFrom;
			if (evolves_from_species) {
				evolvesFrom = {
					name: toTitleCase(evolves_from_species.name, " "),
					url: evolves_from_species.url.split("/")[evolves_from_species.url.split("/").length - 2]
				};
			} else {
				evolvesFrom = undefined;
			}

			const formattedStats = stats.map((stat) => {
				return {
					name: toTitleCase(stat.stat.name, "-"),
					base_stat: stat.base_stat
				};
			});

			// Species Variety
			const species = genera.filter((g) => g.language.name === "en")[0].genus;

			return {
				name: toTitleCase(name, " "),
				types: types.map((type) => ({
					name: toTitleCase(type.type.name, " "),
					color: TYPE_COLORS[type.type.name.toLowerCase()]
				})),
				description: getFlavorText(flavor_text_entries),
				sprites,
				stats: formattedStats,
				height,
				weight,
				captureRate: capture_rate,
				growthRate: toTitleCase(growth_rate.name, "-"),
				genderRatio,
				eggGroups,
				abilities: formattedAbilities,
				evs,
				evolvesFrom,
				pokemonTheme,
				species
			};
		} catch (error) {
			throw new Error(error);
		}
	}

	const dispatch = createEventDispatcher();

	// I didn't have to do it like this, but I wanted to use this feature!
	function destroySelf() {
		dispatch("destroyModal", {
			hideModal: true
		});
	}
</script>

<div class="modal-container">
	<div class="modal-inner">
		<button on:click={destroySelf}>X</button>
		{#await pokemonDetails}
			LOADING...
		{:then details}
			<!-- <pre>{JSON.stringify(details, null, 2)}</pre> -->
			<header>
				<span>
					<h1>{details.name}</h1>
					{#each details.types as type (type.name)}
						<span class="type" style="background-color: {type.color}">
							{type.name}
						</span>
					{/each}
				</span>
			</header>

			<div>
				<h2>{details.description}</h2>

				<div>
					<PokemonSprites sprites={details.sprites} />
				</div>

				<div>
					<div class="stats">
						<section>
							<h2>Profile</h2>
							<dl>
								<dt>Species:</dt>
								<dd>{details.species}</dd>
								<dt>Height:</dt>
								<dd>
									{details.height / 10} m ({Math.round(
										(details.height * 0.328084 + 0.00001) * 100
									) / 100}
									lb)
								</dd>
								<dt>Weight:</dt>
								<dd>
									{details.weight / 10} kg ({Math.round(
										(details.weight * 0.220462 + 0.00001) * 100
									) / 100}
									lb)
								</dd>
								<dt>Abilities:</dt>
								<dd>{details.abilities}</dd>
							</dl>
						</section>

						<section>
							<h2>Training Stats</h2>
							<dl>
								<dt>EV Yield:</dt>
								<dd>{details.evs}</dd>
								<dt>Capture Rate:</dt>
								<dd>{details.captureRate}</dd>
								<dt>Growth Rate:</dt>
								<dd>{details.growthRate}</dd>
							</dl>
						</section>

						<section>
							<h2>Breeding Stats</h2>
							<dl>
								<div>
									<dt>Gender Ratio (%F / %M):</dt>
									<div>
										<dd>
											<DataBar value={details.genderRatio.female} statColor={femaleColor} />
											<DataBar value={details.genderRatio.male} statColor={maleColor} />
										</dd>
									</div>
								</div>
								<dt>Egg Groups:</dt>
								<dd>
									{#each details.eggGroups as egg}{egg}{/each}
								</dd>
							</dl>
						</section>

						<section>
							<h2>Base Stats</h2>
							<dl>
								{#each details.stats as stat}
									<div>
										<dt>{stat.name}</dt>
										<dd>
											<DataBar value={stat.base_stat} statColor={details.pokemonTheme} />
										</dd>
									</div>
								{/each}
							</dl>
						</section>
					</div>
				</div>
			</div>

			<footer>
				{#if details.evolvesFrom}
					<h2>
						{details.name} evolves from {details.evolvesFrom.name}
					</h2>
				{:else}
					<h2>{details.name} does not evolve from another Pokemon</h2>
				{/if}
			</footer>
		{:catch error}
			<p style="color: red">{error.message}</p>
		{/await}
	</div>
</div>

<style>
	header {
		display: flex;
		flex-direction: row;
	}

	h2 {
		font-weight: bold;
	}

	button {
		color: grey;
		background-color: transparent;
		border-color: transparent;
		float: right;
		font-size: 28px;
		font-weight: bold;
	}

	button:hover,
	button:focus {
		color: red;
		text-decoration: none;
		cursor: pointer;
	}

	.modal-container {
		position: fixed;
		z-index: 1;
		border-radius: 5%;
		left: 5%;
		top: 5%;
		height: 90%;
		width: 90%;
		overflow-y: auto;
		background-color: aliceblue;
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
	}

	.modal-inner {
		padding: 1%;
	}

	.stats {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr 1fr;
		border-style: groove;
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
	}

	.type {
		color: aliceblue;
		border-radius: 15% 0 15% 0;
		padding: 2px;
	}
</style>
