<script>
	import { goto } from '$app/navigation';

	import { page } from '$app/stores';
	import API from '$lib/api/api';
	import { onMount } from 'svelte';

    let recipe;
    onMount(async () => {
        recipe = await API.get(`/recipes/${$page.params.id}.json`)
    })
	import Recipe from '$lib/components/Recipes/Recipe/Flow.svelte';
</script>

<div class="btn btn-warning" on:click={() => goto(`/recipes/${recipe.id}/edit`)}>Edit</div>
<br />
{#if recipe}
    <Recipe {recipe}></Recipe>
{/if}

<style></style>
