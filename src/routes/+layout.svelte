<script lang="ts">
	import '../app.css';
	import Background from '$lib/Background.svelte';
	let { children } = $props();
</script>

<div class="layout-wrapper">
	<Background />
	<div class="content-wrapper">
		{@render children()}
	</div>
</div>

<style>
	.layout-wrapper {
		position: relative;
		min-height: 100vh;
		overflow: hidden;
	}

	.content-wrapper {
		position: relative;
		z-index: 1;
	}

	:global(body) {
		margin: 0;
		overflow: hidden;
	}
</style>
