<script>
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';
import { onMount } from 'svelte';

	const progress = tweened(0, {
		duration: 2500,
		easing: cubicOut
	});

    onMount(() => progress.set(1))
</script>

<progress value={$progress}></progress>

<style>
	progress {
		display: block;
		width: 100%;
	}
</style>