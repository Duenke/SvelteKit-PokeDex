<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";

	export let pokedexId;
	export let pokemonName;

	const dispatch = createEventDispatcher();

	let pokemonDetails;
	let loading = false;
	let error;

	$: getPokemon(pokedexId); // a hack version of onMount()

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

	function getFlavorText(flavors) {
		// RANDOM FLAVOR TEXT
		let tempDescription = [];
		tempDescription = flavors
			.filter((flavor) => flavor.language.name === "en")
			.map((item) => item.flavor_text);

		const num = Math.floor(Math.random() * tempDescription.length);

		return tempDescription[num];
	}

	async function getPokemon(pokedexId) {
		const pokemonURL = `https://pokeapi.co/api/v2/pokemon/${pokedexId}/`;
		const pokemonSpeciesURL = `https://pokeapi.co/api/v2/pokemon-species/${pokedexId}/`;

		try {
			loading = true;
			const pokemonGeneral = await ky.get(pokemonURL).json();
			const pokemonSpecies = await ky.get(pokemonSpeciesURL).json();

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

			loading = false;

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
		} catch (error) {
			loading = false;
			error = error;
		}
	}

	function destroySelf() {
		dispatch("destroyModal", {
			hideModal: true
		});
	}

	// onMount((pokemonDetails = getPokemonDetails()));
</script>

<div class="modal-container">
	<button on:click={destroySelf}>X</button>
	<div class="border border-gray-500 rounded-lg overflow-hidden">
		<!-- <pre>{JSON.stringify(pokemon, null, 2)}</pre> -->
		<header class="flex justify-between p-4 bg-gray-200 border-b border-gray-500">
			<h1 class="text-lg">{pokemonName}</h1>
			<div>
				{#each pokemonDetails.types as type (type.name)}
					<span
						class="inline-block ml-1 py-1 px-2 text-xs text-white tracking-wide
                rounded-full bg-{type.color}"
					>
						{type.name}
					</span>
				{/each}
			</div>
		</header>
        {@debug pokemonDetails}
		<div class="p-4">
			<p>{pokemonDetails.description}</p>

			<div class="grid grid-cols-8 gap-2">
				<!-- <PokemonSprites sprites={pokemonDetails.sprites} /> -->
			</div>

			<div class="flex flex-col lg:flex-row mt-6">
				<div class="w-full lg:w-7/12">
					<section>
						<h2 class="text-lg font-bold">Profile</h2>
						<dl class="flex flex-wrap text-sm">
							<dt class="w-5/12">Species:</dt>
							<dd class="flex-grow w-7/12">{pokemonDetails.species}</dd>
							<dt class="w-5/12">Height:</dt>
							<dd class="flex-grow w-7/12">
								{pokemonDetails.height / 10} m ({Math.round(
									(pokemonDetails.height * 0.328084 + 0.00001) * 100
								) / 100}
								lb)
							</dd>
							<dt class="w-5/12">Weight:</dt>
							<dd class="flex-grow w-7/12">
								{pokemonDetails.weight / 10} kg ({Math.round(
									(pokemonDetails.weight * 0.220462 + 0.00001) * 100
								) / 100}
								lb)
							</dd>
							<dt class="w-5/12">Abilities:</dt>
							<dd class="flex-grow w-7/12">{pokemonDetails.abilities}</dd>
						</dl>
					</section>

					<section class="mt-6">
						<h2 class="text-lg font-bold">Training Stats</h2>
						<dl class="flex flex-wrap text-sm">
							<dt class="w-5/12">EV Yield:</dt>
							<dd class="flex-grow w-7/12">{pokemonDetails.evs}</dd>
							<dt class="w-5/12">Capture Rate:</dt>
							<dd class="flex-grow w-7/12">{pokemonDetails.captureRate}</dd>
							<dt class="w-5/12">Growth Rate:</dt>
							<dd class="flex-grow w-7/12">{pokemonDetails.growthRate}</dd>
						</dl>
					</section>

					<section class="mt-6">
						<h2 class="text-lg font-bold">Breeding Stats</h2>
						<dl class="flex flex-wrap text-sm">
							<div class="flex items-center justify-center w-full">
								<dt class="w-5/12">Gender Ratio (%F / %M):</dt>
								<div class="w-7/12">
									<dd class="flex flex-grow h-4">
										<!-- <DataBar
                          value={pokemonDetails.genderRatio.female}
                          statColor={femaleColor}
                        />
                        <DataBar
                          value={pokemonDetails.genderRatio.male}
                          statColor={maleColor}
                        /> -->
									</dd>
								</div>
							</div>
							<dt class="w-5/12">Egg Groups:</dt>
							<dd class="flex-grow w-7/12">
								{#each pokemonDetails.eggGroups as egg}{egg}{/each}
							</dd>
						</dl>
					</section>
				</div>

				<div class="w-full lg:w-5/12">
					<section class="mt-6 lg:mt-0">
						<h2 class="text-lg font-bold">Base Stats</h2>
						<dl>
							{#each pokemonDetails.stats as stat}
								<div class="flex items-center">
									<dt class="w-4/12">{stat.name}</dt>
									<dd class="w-7/12 h-4 text-sm bg-gray-200 rounded">
										<!-- <DataBar
                          value={stat.base_stat}
                          statColor={pokemonDetails.pokemonTheme}
                        /> -->
									</dd>
								</div>
							{/each}
						</dl>
					</section>
				</div>
			</div>
		</div>

		<footer class="flex justify-center w-full p-4 border-t border-gray-500">
			{#if pokemonDetails.evolvesFrom}
				<p>
					{pokemonName} evolves from
					<!-- <Link to="/{pokemonDetails.evolvesFrom.url}">
                <span class="underline">{pokemonDetails.evolvesFrom.name}</span>
              </Link> -->
				</p>
			{:else}
				<p>{pokemonName} does not evolve from another Pokemon</p>
			{/if}
		</footer>
	</div>
</div>

<style>
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
		background-color: aliceblue;
	}
</style>
