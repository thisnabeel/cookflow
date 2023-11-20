<script>
    import Steps from "./Steps/Index.svelte"
    export let recipe;
</script>


<input type="text" class="form-control title" bind:value={recipe.title}/>
<hr>
<Steps {recipe}></Steps>

<style>
    .title {
        font-size: 34px;
        text-align: center;
        padding: 0.5em;
    }
</style>