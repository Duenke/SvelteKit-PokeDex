<script>
	import { tweened } from "svelte/motion";

	export let statColor;
	export let value;

	let amount = tweened(0);
	$: amount.set(value > 100 ? 100 : value);
</script>

<div style="width: {$amount}%; background-color: {statColor};">
	<span>{value}</span>
</div>

<style>
	div {
		border-radius: 25px;
		padding: 2px;
		color: aliceblue;
	}
</style>
