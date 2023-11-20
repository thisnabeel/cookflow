<script>
	import { onMount, afterUpdate } from 'svelte';
	// import Nav from './Nav/Index.svelte';
	import API from '$lib/api/api';
	// import SelectedRecipe from './Recipe/Edit.svelte';
	import ListItem from './ListItem.svelte';
	import debounceSave from '$lib/functions/debounce';
	export let editable = false;
	let recipes = [];
	let selectedRecipe = null;
	let visitingRecipe = null;
	let organizedRecipes = [];

	let unsaved = false;
	let saving = false;

	let newTitle = '';

	async function create() {
		if (newTitle.length < 3) return;
		const res = await API.post('/recipes.json', {
			title: newTitle,
			position: recipes.length
		});
		recipes = [...recipes, res];
		organizedRecipes = organizeRecipes(recipes);
		newTitle = '';
	}

	onMount(async () => {
		const res = await API.get('/recipes.json');
		recipes = res;
		console.log('first recipe fetch', recipes);
		organizedRecipes = organizeRecipes(recipes);
		console.log('organized recipes', organizedRecipes);
	});

	afterUpdate(() => {
		// Add event listeners for arrow key navigation
		window.addEventListener('keydown', handleKeyDown);

		return () => {
			// Remove event listeners on component destruction
			window.removeEventListener('keydown', handleKeyDown);
		};
	});

	afterUpdate(() => {
		// Add event listeners for arrow key navigation
		window.addEventListener('keydown', handleKeyDown);

		return () => {
			// Remove event listeners on component destruction
			window.removeEventListener('keydown', handleKeyDown);
		};
	});

	function handleKeyDown(event) {
		if (!selectedRecipe || !organizedRecipes) return;

		let index = -1;
		let siblings = findSiblings(organizedRecipes, selectedRecipe.id);

		if (selectedRecipe.recipe_id) {
			index = siblings.findIndex((recipe) => recipe.id === selectedRecipe.id);
		} else {
			siblings = organizedRecipes;
			index = organizedRecipes.findIndex((recipe) => recipe.id === selectedRecipe.id);
		}

		if (event.key === 's' || event.key === 'w') {
			handleUpDownKey(event, siblings, index);
		} else if (event.key === 'd') {
			handleRightKey(siblings, index);
		} else if (event.key === 'a') {
			handleLeftKey(siblings, index);
		}

		organizedRecipes = organizedRecipes.map((recipe) => {
			if (recipe.id === selectedRecipe.id && selectedRecipe.recipe_id) {
				return {
					...recipe,
					children: [...siblings]
				};
			}
			return recipe;
		});

		unsaved = true;

		// saveOrder(organizedRecipes);
		// saveOrder(updatePositions(organizedRecipes));
	}

	async function saveOrder() {
		// console.log({ array });
		saving = true;
		const array = updatePositions(organizedRecipes);
		await debounceSave('/recipes/reorder', {
			list: flattenTree(array).map((g) => {
				return { id: g.id, recipe_id: g.recipe_id, position: g.position };
			})
		});
		saving = false;
		unsaved = false;
	}

	function updatePositions(array) {
		// console.log('GOTTEN', array);
		if (!array) return;
		for (let i = 0; i < array.length; i++) {
			array[i].position = i + 1;
			array[i].children = updatePositions(array[i].children);
		}

		return array;
	}

	function handleUpDownKey(event, container, index) {
		if (selectedRecipe) {
			if (!container) {
				console.log('no parent');
			} else {
				console.log({ container });
			}

			if (event.key === 's' && index < container.length - 1) {
				// Move the recipe down
				swapPositions(container, index, index + 1);
			} else if (event.key === 'w' && index > 0) {
				// Move the recipe up
				swapPositions(container, index, index - 1);
			}
		}
	}

	function handleRightKey(siblings, index) {
		const currentIndex = siblings.indexOf(selectedRecipe);
		const futureParentIndex = currentIndex - 1;

		console.log({ futureParentIndex });

		if (futureParentIndex >= 0 && futureParentIndex < siblings.length - 1) {
			// Save the removed element
			const removedElement = siblings.splice(currentIndex, 1)[0];

			selectedRecipe.recipe_id = siblings[futureParentIndex].id;
			if (!siblings[futureParentIndex].children) {
				siblings[futureParentIndex].children = [];
			}
			siblings[futureParentIndex].children = [
				...siblings[futureParentIndex].children,
				removedElement
			];
		}
	}

	function handleLeftKey(siblings, index) {
		if (!selectedRecipe || !selectedRecipe.recipe_id) {
			return;
		} else {
			const grandparentId = findGrandparentId(organizedRecipes, selectedRecipe.recipe_id);
			let newParent;
			let currentParent;
			let currentParentIndex;
			if (grandparentId !== null) {
				newParent = findRecipe(organizedRecipes, grandparentId);
				currentParent = findRecipe(organizedRecipes, selectedRecipe.recipe_id);
				currentParentIndex = newParent.children.findIndex(
					(recipe) => recipe.id === currentParent.id
				);

				console.log({ newParent });
				console.log({ currentParent });
				console.log({ currentParentIndex });
			} else {
				newParent = organizedRecipes;
				currentParent = findRecipe(organizedRecipes, selectedRecipe.recipe_id);
				currentParentIndex = newParent.findIndex((recipe) => recipe.id === currentParent.id);
			}

			if (currentParentIndex >= 0) {
				// Remove the selected recipe from its parent
				const removedElement = currentParent.children.splice(index, 1)[0];
				console.log(currentParent.children);
				removedElement.recipe_id = newParent.id;
				if (Array.isArray(newParent)) {
					newParent.splice(currentParentIndex + 1, 0, removedElement);
				} else {
					newParent.children.splice(currentParentIndex + 1, 0, removedElement);
				}
				return;
			}
			return;
		}
	}

	function swapPositions(array, indexA, indexB) {
		const temp = array[indexA];
		array[indexA] = array[indexB];
		array[indexB] = temp;
	}

	function flattenTree(tree) {
		return tree.reduce((acc, node) => {
			acc.push(node);
			if (node.children) {
				acc.push(...flattenTree(node.children));
			}
			return acc;
		}, []);
	}
	function findGrandparentId(arr, recipeId) {
		for (let recipe of arr) {
			if (recipe.children && recipe.children.some((child) => child.id === recipeId)) {
				return recipe.id;
			}
			if (recipe.children) {
				const result = findGrandparentId(recipe.children, recipeId);
				if (result !== null) {
					return result; // Return if found in children
				}
			}
		}
		return null;
	}

	function findRecipe(arr, recipeId) {
		// console.log({ recipeId });
		for (let recipe of arr) {
			if (recipe.id === recipeId) {
				return recipe;
			}
			if (recipe.children) {
				const result = findRecipe(recipe.children, recipeId);
				if (result !== null) {
					return result; // Return if found in children
				}
			}
		}
		return null;
	}
	// function findRecipeIndex(arr, recipeId) {
	// 	for (let recipe of arr) {
	// 		// console.log({ recipe }, recipeId);
	// 		if (recipe.id === recipeId) {
	// 			return recipe;
	// 		}
	// 		const result = findParent(recipe.children || [], recipeId);
	// 	}
	// 	return recipe;
	// }

	function findSiblings(arr, recipeId) {
		for (let recipe of arr) {
			if (recipe.children && recipe.children.some((child) => child.id === recipeId)) {
				return recipe.children;
			}
			if (recipe.children) {
				const result = findSiblings(recipe.children, recipeId);
				if (result.length > 0) {
					console.log('FOUND IN CHILDREN');
					return result; // Return if found in children
				}
			}
		}
		return [];
	}

	// $: organizedRecipes = organizeRecipes(recipes);
	// $: console.log({ organizedRecipes });

	function organizeRecipes(recipes, parentId = null) {
		// Filter recipes based on the current parent ID
		const filteredRecipes = recipes.filter((recipe) => recipe.recipe_id === parentId);

		// Recursively organize children for each filtered recipe
		const organizedChildren = filteredRecipes.map((parentRecipe) => {
			const children = organizeRecipes(recipes, parentRecipe.id);
			return { ...parentRecipe, children };
		});

		return organizedChildren;
	}

	function selectRecipe(recipe) {
		selectedRecipe = recipe;
		console.log({ selectedRecipe });
	}

	function visitRecipe(recipe) {
		visitingRecipe = recipe;
		console.log({ visitingRecipe });
	}
</script>

<div class="content">
	{#if editable}
		<div class="form">
			<input
				type="text"
				class="form-control"
				bind:value={newTitle}
				placeholder="Enter Title For New Recipe..."
			/>
			<div class="btn btn-outline-primary" on:click={create}>Add</div>
		</div>
	{/if}

	<!-- <Nav /> -->

	<div class="wrapper">
		<ul class="clean-list recipes">
			{#each organizedRecipes || [] as recipe}
				<ListItem {editable} {selectedRecipe} {recipe} {selectRecipe} {visitRecipe} />
			{/each}
		</ul>
	</div>

	<div class="saveProgress">
		{#if unsaved && !saving}
			<div class="btn btn-warning" on:click={saveOrder}>Unsaved Progress</div>
		{:else}
			<div class="btn btn-success"><i class="fa fa-check" /></div>
		{/if}

		{#if saving}
			<div class="btn btn-info">Saving</div>
		{/if}
	</div>
</div>

<style>
	.saveProgress {
		position: fixed;
		bottom: 10px;
		right: 10px;
	}
	/* ... (your existing styles) */
	.selected {
		background-color: #b8eeff !important;
	}

	.wrapper {
		padding: 20px;
	}

	.recipe {
		padding: 8px;
		font-size: 24px;
		background: rgb(255 248 222);
		margin: 4px;
		border-radius: 8px;
	}
</style>
