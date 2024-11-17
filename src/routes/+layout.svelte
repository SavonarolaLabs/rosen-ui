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
		background-image: url('https://i.pinimg.com/originals/73/6c/a3/736ca32dff9564696d7d5daad75d1954.jpg');
		background-position: center;
		background-size: cover;
	}
</style>
