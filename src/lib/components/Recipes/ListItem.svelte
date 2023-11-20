<script>
	import { goto } from '$app/navigation';
	import ListItem from './ListItem.svelte';

	export let recipe;
	export let selectedRecipe;
	export let selectRecipe;
	export let visitRecipe;
	export let editable = false;
</script>

<li class="recipe" class:selected={selectedRecipe ? selectedRecipe.id === recipe.id : false}>
	<div class="head" on:click={() => selectRecipe(recipe)}>
		{recipe.title}

		<div
			class="btn btn-warning"
			on:click={() => {
				goto(`/recipes/${recipe.id}/edit`);
			}}
		>
			<i class="fa fa-link" />
		</div>

		<div
			class="btn btn-outline-primary"
			on:click={() => {
				goto(`/recipes/${recipe.id}`);
			}}
		>
			<i class="fa fa-eye" />
		</div>
	</div>

	<div class="children">
		{#each recipe.children || [] as child}
			<ListItem {editable} {selectedRecipe} recipe={child} {selectRecipe} {visitRecipe} />
		{/each}
	</div>
</li>

<style>
	.selected > .head {
		background-color: #b8eeff !important;
	}

	.wrapper {
		padding: 20px;
	}

	.recipe {
		padding-left: 10px;
	}
	.recipe.selected {
		padding-left: 6px;
		border-left: 4px solid #b8eeff;
	}

	.head {
		padding: 8px;
		font-size: 24px;
		background: rgb(255 248 222);
		margin: 4px;
		border-radius: 8px;
	}

	.children {
		margin-left: 32px;
	}
</style>
