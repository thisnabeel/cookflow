<script>
	import API from "$lib/api/api";
	import { onMount } from "svelte";

    export let recipe;
    let steps = [];
    let organizedSteps = [];
    let level = 0;
    let percentage = 0;
	onMount(async () => {

        if (!recipe) return;
		const res = await API.get(`/recipes/${recipe.id}/steps.json`);
		steps = res.steps;
		console.log('first step fetch', steps);
		// organizedSteps = organizeSteps(steps);
		console.log('organized steps', organizedSteps);
	});
    
    $: currentStep =  steps && steps[level] ? steps[level] : null;

    $: {
        const total = steps.length
		const number = level + 1
		const result = number * 100 / total;

	    percentage = Math.round(result.toFixed(2));

    }
</script>


{#if recipe}
    <h1>
        {recipe.title}
    </h1>
    {#if currentStep}
        <div class="step">
            {currentStep.title}
        </div>
    {/if}
        


        <div class="bottom">

            <div class="progress">
            <div class="progress-bar" role="progressbar" 
            aria-valuemin="0" aria-valuemax="100" style="width:{percentage}%">
            </div>
            </div>
            <p class="percentage text-center">{percentage}%</p>

            <div class="flex flow-nav">
        
                    {#if level > 0}
                        <div class="btn btn-danger" on:click={() => level -= 1}>Previous</div>
                    {/if}
        
        
                    {#if level < steps.length - 1}
                        <div class="btn btn-primary" on:click={() => level += 1}>Next</div>
                    {/if}
        
            </div>
        </div>
{/if}

<style>
    .flex {
        display: flex;
    }

    .flex > .btn {
        font-size: 24px;
        flex: 1 1 50%;
    }

    .bottom {
        position: fixed;
        bottom: 10px;
        width: 100%;
        left: 0;
    }

    .step {
        padding: 8px;
        font-size: 24px;
        background: rgb(255 248 222);
        margin: 4px;
        border-radius: 8px;
    }
</style>