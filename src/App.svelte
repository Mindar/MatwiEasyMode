<script lang="ts">
import type { ComponentType, SvelteComponent } from "svelte";
import AbcAnalysis from "./lib/AbcAnalysis.svelte";


type ToolComponent = AbcAnalysis;

interface ToolType {
	title: string;
	component: ComponentType<ToolComponent>;
}

function showComponent(component: ComponentType | null) {
	console.log(component);
	toolSelected = component;
}

const tools = [{
	title: "ABC Analyse",
	component: AbcAnalysis
}];


let toolSelected: ComponentType | null = null;

</script>

{#if toolSelected != null}
	<svelte:component this={toolSelected} on:goBack={() => showComponent(null)} />
{:else}
	<main>
		{#each tools as tool, index(index)}
			<div on:click={() => showComponent(tool.component)}>
				{tool.title}
			</div>
		{/each}
	</main>
{/if}

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	div {
		padding: 1rem;
		margin: 1rem;
		border: 1px solid black;
		border-radius: 0.5rem;
	}
</style>
