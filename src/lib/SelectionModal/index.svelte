<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";

	export let pokedexId;

	const dispatch = createEventDispatcher();

	let pokemonDetails;
	let error;

	const TYPE_COLORS = {
		bug: "green-200",
		dark: "orange-900",
		dragon: "purple-600",
		electric: "orange-400",
		fairy: "pink-300",
		fighting: "orange-800",
		fire: "red-600",
		flying: "purple-400",
		ghost: "purple-700",
		grass: "green-400",
		ground: "yellow-500",
		ice: "teal-200",
		normal: "orange-300",
		poison: "purple-800",
		psychic: "red-500",
		rock: "yellow-700",
		steel: "gray-400",
		water: "blue-600"
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

			const pokemonSpeciesResponse = await fetch(pokemonSpeciesURL);
			const pokemonSpecies = await pokemonSpeciesResponse.json();

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

			pokemonDetails = {
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

			return pokemonDetails;
		} catch (error) {
			error = error;
		}
	}

	function destroySelf() {
		dispatch("destroyModal", {
			hideModal: true
		});
	}

	onMount(() => pokemonDetails = getPokemonDetails(pokedexId));
</script>

<div class="modal-container">
	<button on:click={destroySelf}>X</button>
	{#await pokemonDetails}
		LOADING...
	{:then pokemonDetails}
		<div>
			<pre>{JSON.stringify(pokemonDetails, null, 2)}</pre>
			<!-- <header>
				<h1>{pokemonDetails.name}</h1>
				<div>
					{#each pokemonDetails.types as type (type.name)}
						<span class={type.color}>
							{type.name}
						</span>
					{/each}
				</div>
			</header>

			<div>
				<p>{pokemonDetails.description}</p>

				<div>
					<PokemonSprites sprites={pokemonDetails.sprites} />
				</div>

				<div>
					<div>
						<section>
							<h2>Profile</h2>
							<dl>
								<dt>Species:</dt>
								<dd>{pokemonDetails.species}</dd>
								<dt>Height:</dt>
								<dd>
									{pokemonDetails.height / 10} m ({Math.round(
										(pokemonDetails.height * 0.328084 + 0.00001) * 100
									) / 100}
									lb)
								</dd>
								<dt>Weight:</dt>
								<dd>
									{pokemonDetails.weight / 10} kg ({Math.round(
										(pokemonDetails.weight * 0.220462 + 0.00001) * 100
									) / 100}
									lb)
								</dd>
								<dt>Abilities:</dt>
								<dd>{pokemonDetails.abilities}</dd>
							</dl>
						</section>

						<section>
							<h2>Training Stats</h2>
							<dl>
								<dt>EV Yield:</dt>
								<dd>{pokemonDetails.evs}</dd>
								<dt>Capture Rate:</dt>
								<dd>{pokemonDetails.captureRate}</dd>
								<dt>Growth Rate:</dt>
								<dd>{pokemonDetails.growthRate}</dd>
							</dl>
						</section>

						<section>
							<h2>Breeding Stats</h2>
							<dl>
								<div>
									<dt>Gender Ratio (%F / %M):</dt>
									<div>
										<dd>
											<DataBar
                          value={pokemonDetails.genderRatio.female}
                          statColor={femaleColor}
                        />
                        <DataBar
                          value={pokemonDetails.genderRatio.male}
                          statColor={maleColor}
                        />
										</dd>
									</div>
								</div>
								<dt>Egg Groups:</dt>
								<dd>
									{#each pokemonDetails.eggGroups as egg}{egg}{/each}
								</dd>
							</dl>
						</section>
					</div>

					<div>
						<section>
							<h2>Base Stats</h2>
							<dl>
								{#each pokemonDetails.stats as stat}
									<div>
										<dt>{stat.name}</dt>
										<dd>
											<DataBar
                          value={stat.base_stat}
                          statColor={pokemonDetails.pokemonTheme}
                        />
										</dd>
									</div>
								{/each}
							</dl>
						</section>
					</div>
				</div>
			</div>

			<footer>
				{#if pokemonDetails.evolvesFrom}
					<p>
						{pokemonDetails.name} evolves from
						<Link to="/{pokemonDetails.evolvesFrom.url}">
                <span class="underline">{pokemonDetails.evolvesFrom.name}</span>
              </Link>
					</p>
				{:else}
					<p>{pokemonDetails.name} does not evolve from another Pokemon</p>
				{/if}
			</footer> -->
		</div>
	{/await}
</div>

<style>
	header {
		display: flex;
		flex-direction: row;
	}
	button {
		color: grey;
		float: right;
		font-size: 28px;
		font-weight: bold;
	}

	button:hover,
	button:focus {
		color: black;
		text-decoration: none;
		cursor: pointer;
	}

	.modal-container {
		border-radius: 5%;
		position: fixed;
		z-index: 1;
		left: 5%;
		top: 5%;
		height: 90%;
		width: 90%;
		overflow-y: auto;
		background-color: aliceblue;
	}
</style>
