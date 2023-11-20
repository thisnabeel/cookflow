<script>
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import API from '$lib/api/api';
	import Recipe from '$lib/components/Recipes/Recipe/Edit.svelte';
	import { onMount } from 'svelte';

    let recipe;
    onMount(async () => {
        recipe = await API.get(`/recipes/${$page.params.id}.json`)
    })
</script>

<h1>Editing Recipe:
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <span on:click={() => goto(`/recipes/${recipe.id}`)} class="btn btn-outline-primary float-end">View Recipe</span>
</h1>

{#if recipe}
    <Recipe {recipe}></Recipe>
{/if}

<style></style>
