<script>
	import { goto } from '$app/navigation';
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Cook Flow" />
</svelte:head>

<section>
	<script>
		import Recipes from '$lib/components/Recipes/Index.svelte';
	</script>

	<h1>Welcome!</h1>
	<p>
		🌟 Welcome to "Umm Sukkar" – your portal to culinary delight! Dive into the art of gastronomy
		with our meticulously curated recipes, marrying sophistication and simplicity. Whether you're a
		seasoned chef or a kitchen explorer, our step-by-step guides ensure an enjoyable culinary
		journey. Join us in celebrating the joy of cooking, where every ingredient and step is a note in
		the grand symphony of flavor. Indulge in the extraordinary at "Umm Sukkar," where culinary
		excellence meets playful flair. 🍽️🎶✨
	</p>

	<div class="btn btn-primary" on:click={() => goto('/recipes')}>View Recipes</div>

	<style></style>
</section>

<style>
</style>
