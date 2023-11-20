<script>
	import { onMount, afterUpdate } from 'svelte';
	// import Nav from './Nav/Index.svelte';
	import API from '$lib/api/api';
	// import SelectedStep from './Step/Edit.svelte';
	import ListItem from './ListItem.svelte';
	import debounceSave from '$lib/functions/debounce';

	export let recipe;
	let steps = [];
	let selectedStep = null;
	let visitingStep = null;
	let organizedSteps = [];
	
	let unsaved = false;
	let saving = false;
	
	let newTitle = '';
	let typing = false;

	async function create() {
		if (newTitle.length < 3) return;
		const res = await API.post('/recipe_steps.json', {
			title: newTitle,
			position: steps.length,
			recipe_id: recipe.id,
			recipe_step_id: selectedStep ? selectedStep.id : null
		});
		steps = [...steps, res];
		organizedSteps = organizeSteps(steps);
		console.log({organizedSteps})
		newTitle = '';
	}

	async function deleteStep(payload) {
		console.log({payload})
		steps = steps.filter(step => step.id !== payload.id);
		await API.delete(`/recipe_steps/${payload.id}.json`);
		organizedSteps = organizeSteps(steps);
		console.log({organizedSteps});
		saveOrder();
	}

	onMount(async () => {
		const res = await API.get(`/recipes/${recipe.id}/steps.json`);
		steps = res.steps;
		console.log('first step fetch', steps);
		organizedSteps = organizeSteps(steps);
		console.log('organized steps', organizedSteps);
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
		if (typing) return;
		if (!selectedStep || !organizedSteps) return;

		let index = -1;
		let siblings = findSiblings(organizedSteps, selectedStep.id);

		if (selectedStep.recipe_step_id) {
			index = siblings.findIndex((step) => step.id === selectedStep.id);
		} else {
			siblings = organizedSteps;
			index = organizedSteps.findIndex((step) => step.id === selectedStep.id);
		}

		if (event.key === 's' || event.key === 'w') {
			handleUpDownKey(event, siblings, index);
		} else if (event.key === 'd') {
			handleRightKey(siblings, index);
		} else if (event.key === 'a') {
			handleLeftKey(siblings, index);
		}

		organizedSteps = organizedSteps.map((step) => {
			if (step.id === selectedStep.id && selectedStep.recipe_step_id) {
				return {
					...step,
					children: [...siblings]
				};
			}
			return step;
		});

		unsaved = true;

		// saveOrder(organizedSteps);
		// saveOrder(updatePositions(organizedSteps));
	}

	async function saveOrder() {
		// console.log({ array });
		saving = true;
		const array = updatePositions(organizedSteps);
		await debounceSave('/recipe_steps/reorder', {
			list: flattenTree(array).map((g) => {
				return { id: g.id, step_id: g.recipe_step_id, position: g.position };
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
		if (selectedStep) {
			if (!container) {
				console.log('no parent');
			} else {
				console.log({ container });
			}

			if (event.key === 's' && index < container.length - 1) {
				// Move the step down
				swapPositions(container, index, index + 1);
			} else if (event.key === 'w' && index > 0) {
				// Move the step up
				swapPositions(container, index, index - 1);
			}
		}
	}

	function handleRightKey(siblings, index) {
		const currentIndex = siblings.indexOf(selectedStep);
		const futureParentIndex = currentIndex - 1;

		console.log({ futureParentIndex });

		if (futureParentIndex >= 0 && futureParentIndex < siblings.length - 1) {
			// Save the removed element
			const removedElement = siblings.splice(currentIndex, 1)[0];

			selectedStep.recipe_step_id = siblings[futureParentIndex].id;
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
		if (!selectedStep || !selectedStep.recipe_step_id) {
			return;
		} else {
			const grandparentId = findGrandparentId(organizedSteps, selectedStep.recipe_step_id);
			let newParent;
			let currentParent;
			let currentParentIndex;
			if (grandparentId !== null) {
				newParent = findStep(organizedSteps, grandparentId);
				currentParent = findStep(organizedSteps, selectedStep.recipe_step_id);
				currentParentIndex = newParent.children.findIndex(
					(step) => step.id === currentParent.id
				);

				console.log({ newParent });
				console.log({ currentParent });
				console.log({ currentParentIndex });
			} else {
				newParent = organizedSteps;
				currentParent = findStep(organizedSteps, selectedStep.recipe_step_id);
				currentParentIndex = newParent.findIndex((step) => step.id === currentParent.id);
			}

			if (currentParentIndex >= 0) {
				// Remove the selected step from its parent
				const removedElement = currentParent.children.splice(index, 1)[0];
				console.log(currentParent.children);
				removedElement.recipe_step_id = newParent.id;
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
	function findGrandparentId(arr, stepId) {
		for (let step of arr) {
			if (step.children && step.children.some((child) => child.id === stepId)) {
				return step.id;
			}
			if (step.children) {
				const result = findGrandparentId(step.children, stepId);
				if (result !== null) {
					return result; // Return if found in children
				}
			}
		}
		return null;
	}

	function findStep(arr, stepId) {
		// console.log({ stepId });
		for (let step of arr) {
			if (step.id === stepId) {
				return step;
			}
			if (step.children) {
				const result = findStep(step.children, stepId);
				if (result !== null) {
					return result; // Return if found in children
				}
			}
		}
		return null;
	}
	// function findStepIndex(arr, stepId) {
	// 	for (let step of arr) {
	// 		// console.log({ step }, stepId);
	// 		if (step.id === stepId) {
	// 			return step;
	// 		}
	// 		const result = findParent(step.children || [], stepId);
	// 	}
	// 	return step;
	// }

	function findSiblings(arr, stepId) {
		for (let step of arr) {
			if (step.children && step.children.some((child) => child.id === stepId)) {
				return step.children;
			}
			if (step.children) {
				const result = findSiblings(step.children, stepId);
				if (result.length > 0) {
					console.log('FOUND IN CHILDREN');
					return result; // Return if found in children
				}
			}
		}
		return [];
	}

	// $: organizedSteps = organizeSteps(steps);
	// $: console.log({ organizedSteps });

	function organizeSteps(steps, parentId = null) {
		// Filter steps based on the current parent ID
		const filteredSteps = steps.filter((step) => step.recipe_step_id === parentId);

		// Recursively organize children for each filtered step
		const organizedChildren = filteredSteps.map((parentStep) => {
			const children = organizeSteps(steps, parentStep.id);
			return { ...parentStep, children };
		});

		return organizedChildren;
	}

	function selectStep(step) {
		selectedStep = step;
		console.log({ selectedStep });
	}

	function visitStep(step) {
		visitingStep = step;
		console.log({ visitingStep });
	}
</script>

<div class="content">
	<div class="form">
		<input type="text" class="form-control" bind:value={newTitle} on:focus={() => typing = true} on:blur={() => typing = false} placeholder="Type New Step..." />
		<div class="btn btn-outline-primary" on:click={create}>Add</div>
	</div>

	<!-- <Nav /> -->

	<div class="wrapper">
		<ul class="clean-list steps">
			{#each organizedSteps || [] as step}
				<ListItem {selectedStep} {step} {selectStep} {visitStep} {deleteStep} />
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

	.step {
		padding: 8px;
		font-size: 24px;
		background: rgb(255 248 222);
		margin: 4px;
		border-radius: 8px;
	}
</style>
