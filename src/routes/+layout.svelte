<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<div class="gradient-wrapper">
	{@render children()}
</div>

<style>
	.gradient-wrapper {
		min-height: 100vh;
		background-color: #2b2b2b;
		/* background-image: linear-gradient(62deg, #8ec5fc 0%, #e0c3fc 100%); */
		background-image: url('https://cdn.culture.ru/images/8b060b55-a31c-5071-a038-41175a6a60be');
	}
</style>
