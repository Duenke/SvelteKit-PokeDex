<script>
	import { tweened } from "svelte/motion";
	import { cubicOut } from "svelte/easing";
	import { onMount } from "svelte";

	export let timeoutIndex;

	const progress = tweened(0, {
		duration: timeoutIndex * 100 + 1000,
		easing: cubicOut
	});

	onMount(() => progress.set(1));
</script>

<progress value={$progress} />

<style>
	progress {
		display: block;
		width: 100%;
	}
</style>
