<script lang="ts">
import { onMount, type ComponentType, type SvelteComponent } from "svelte";
import AbcAnalysis from "./lib/AbcAnalysis.svelte";
import BomGenerator from "./lib/BomGenerator.svelte";
import Miscellaneous from "./lib/Miscellaneous.svelte";


type ToolComponent = AbcAnalysis | Miscellaneous | BomGenerator;

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
		title: "Stücklisten",
		component: BomGenerator
	},{
		title: "Sonstige Rechner",
		component: Miscellaneous
	}
];


let toolSelected: ComponentType | null = null;


let warningVisible = true;
function toggleWarning(event: KeyboardEvent) {
	if (event.key === "Escape") {
		warningVisible = !warningVisible;
	}
}

</script>

<svelte:window on:keydown={toggleWarning} />

{#if warningVisible}
	<header class="flex-col">
		<span>
			Achtung: Keine Garantie für korrekte Ergebnisse oder sonst irgendetwas! Ergebnisse können falsch sein und müssen unbedingt von Hand nachgerechnet werden. Nutzung ausschließlich für Übungen und nur auf eigene Gefahr.
		</span>
		<br>
		<span class="smol">
			Escape-Taste zum ausblenden dieses Textes drücken oder auf Schließen klicken.
		</span>
		<button class="textbtn" on:click={() => {warningVisible = false}}>Schließen</button>
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
	<p class="pa-30">
		Hinweis: Alle Kommawerte bitte in der englischen Schreibweise, also mit PUNKT statt KOMMA eingeben, sonst kommt Müll raus. Außerdem keine Tausender Trennzeichen verwenden. Also z.B. den Wert 12.345.000,29 so eingeben: <span class="monospace bg-grey">12345000.29</span>
	</p>
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
