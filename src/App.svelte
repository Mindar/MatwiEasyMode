<script lang="ts">
import { onMount, type ComponentType, type SvelteComponent } from "svelte";
import AbcAnalysis from "./lib/AbcAnalysis.svelte";
import XyzAnalysis from "./lib/XyzAnalysis.svelte";
import BomGenerator from "./lib/BomGenerator.svelte";


type ToolComponent = AbcAnalysis | XyzAnalysis | BomGenerator;

interface ToolType {
	title: string;
	component: ComponentType<ToolComponent>;
}

function showComponent(component: ComponentType | null) {
	console.log(component);
	toolSelected = component;
}

const tools = [
	{
		title: "ABC Analyse",
		component: AbcAnalysis
	},{
		title: "XYZ Analyse",
		component: XyzAnalysis
	},{
		title: "Stücklisten",
		component: BomGenerator
	}
];


let toolSelected: ComponentType | null = null;


let warningVisible = true;
function toggleWarning(event: KeyboardEvent) {
	console.log(event);
	if (event.key === "Escape") {
		warningVisible = !warningVisible;
	}
}

</script>

<svelte:window on:keydown={toggleWarning} />

{#if warningVisible}
	<header>
		Achtung: Keine Garantie für korrekte Ergebnisse oder sonst irgendetwas! Ergebnisse können falsch sein und müssen unbedingt von Hand nachgerechnet werden. Nutzung ausschließlich für Übungen und nur auf eigene Gefahr.
		<br>
		<span class="smol">Escape-Taste zum ausblenden dieses Textes drücken.</span>
	</header>
{/if}

{#if toolSelected != null}
	<svelte:component this={toolSelected} on:goBack={() => showComponent(null)} />
{:else}
	<main>
		{#each tools as tool, index(index)}
			<div class="toolbtn" on:click={() => showComponent(tool.component)}>
				{tool.title}
			</div>
		{/each}
		<a class="toolbtn" href="index.html" download>
			Software Downloaden
		</a>
	</main>
{/if}

<style>
/* use header as big fat warning label on top of page */
header {
	background-color: #fcaeae;
	padding: 1rem;
	text-align: center;
	font-size: 1.3em;

}
main {
	display: flex;
	flex-direction: row;
	flex-wrap: wrap;
	align-items: center;
	justify-content: center;
}
a, div {
	padding: 1rem;
	margin: 1rem;
	border: 1px solid black;
	border-radius: 0.5rem;
}

.toolbtn {
	cursor: pointer;
}

.toolbtn:hover {
	background-color: #bdbdbd;
}

.smol {
	font-size: 0.5em;
}
</style>
