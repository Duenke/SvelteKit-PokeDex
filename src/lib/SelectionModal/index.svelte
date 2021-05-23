<script>
	export let hideModal;
	export let pokemon;

	const blacklistProps = ["moves", "sprites", "game_indices"];
</script>

<div hidden={hideModal}>
	<button on:click={() => hideModal = !hideModal}>X</button>
	<ul>
		{#each Object.entries(pokemon) as [key, value]}
			{#if !blacklistProps.includes(key) && typeof value !== "string"}
				<li>{key}: {JSON.stringify(value)}</li>
			{:else if !blacklistProps.includes(key)}
				<li>{key}: {value}</li>
			{/if}
		{/each}
	</ul>
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

	div {
		position: fixed;
		z-index: 1;
		left: 5%;
		top: 5%;
		height: 90%;
		width: 90%;
		background-color: aliceblue;
	}
</style>
