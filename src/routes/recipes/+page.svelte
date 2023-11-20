<script>
	import { goto } from '$app/navigation';
	import Recipes from '$lib/components/Recipes/Index.svelte';
</script>

<div class="btn btn-warning" on:click={() => goto('/recipes/edit')}>Edit</div>
<br />
<Recipes editable={false}></Recipes>

<style></style>
