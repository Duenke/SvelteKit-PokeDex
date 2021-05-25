<script>
	import { tweened } from "svelte/motion";
	import { cubicOut } from "svelte/easing";
	import { onMount } from "svelte";

	export let loadTime;

	const progress = tweened(0, {
		duration: loadTime,
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
