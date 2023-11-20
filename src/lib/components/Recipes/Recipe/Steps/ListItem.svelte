<script>
	import { goto } from '$app/navigation';
	import ListItem from './ListItem.svelte';

	export let step;
	export let selectedStep;
	export let selectStep;
	export let visitStep;
	export let deleteStep;

	$: selected = selectedStep ? selectedStep.id === step.id : false
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<li class="step" class:selected={selected}>
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div class="head" on:click={() => selectStep(step)}>
		{step.title}

	</div>
	{#if selected}
		<div class="btn btn-outline-danger delete-step" on:click={() => deleteStep(step)}><i class="fa fa-trash"></i></div>
	{/if}

	<div class="children">
		{#each step.children || [] as child}
			<ListItem {selectedStep} step={child} {selectStep} {visitStep} {deleteStep} />
		{/each}
	</div>
</li>

<style>

	.delete-step {
		position: absolute;
		right: -46px;
		top: 10px;
	}
	.selected > .head {
		background-color: #b8eeff !important;

	}



	.wrapper {
		padding: 20px;
	}

	.step {
		padding-left: 10px;
		position: relative;
	}
	.step.selected {
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
