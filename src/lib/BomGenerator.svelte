<script lang="ts">
import { createEventDispatcher } from "svelte";
import { instance } from "@viz-js/viz";
    import Dispotable from "./Dispotable.svelte";


const dispatch = createEventDispatcher();

function goBack() {
	dispatch("goBack");
}

let errmsg: string | null = null;

let bomtext = "FE: 2 A, 5 B, 10 q\nA: 3 r, 3 B\nB: 5 t, 4 s";

let components: Map<string, Map<string, number>> = parseComponents(bomtext);
let root: string | null = findRoot(components);
let componentsCumulative: Map<string, number> = accumulateComponents(components, root, 1);

// helper variable to keep track of timeslots more easily
let numTimeslots = 0;


function parseComponents(text: string): Map<string, Map<string, number>>{
	const components = new Map<string, Map<string, number>>();
	const lines = text.split("\n");
	for(const line of lines){

		if(!line.match(/^\s*([A-Za-z_]+):\s?(?:([0-9]+)\s?([A-Za-z_]+))(?:\s?,\s?([0-9]+)\s?([A-Za-z_]+))*\s*$/)) {
			errmsg = "Fehler bei Zeile \"" + line + "\"\n";
			return new Map<string, Map<string, number>>();
		}

		const [name, rest] = line.split(":");
		const parts = rest.split(",");
		const componentsOfPart = new Map<string, number>();
		for(const part of parts){
			const [amount, component] = part.trim().split(" ");
			componentsOfPart.set(component, parseInt(amount));
		}
		components.set(name, componentsOfPart);
	}

	errmsg = null;
	return components;
}

function findRoot(components: Map<string, Map<string, number>>): string | null {
	const allComponents = new Set<string>();
	for(const [name, parts] of components){
		allComponents.add(name);
		for(const [part, amount] of parts){
			allComponents.add(part);
		}
	}
	for(const [name, parts] of components){
		// remove all parts from allComponents
		for(const [part, amount] of parts){
			allComponents.delete(part);
		}
	}

	if(allComponents.size !== 1){
		return null;
	}

	return allComponents.values().next().value;
}

function accumulateComponents(components: Map<string, Map<string, number>>, root: string | null, amount: number): Map<string, number> {
	let result = new Map<string, number>();
	
	if(root === null){
		return result;
	}

	// If the root is not in the components map, it's a leaf node and we can just return the amount
	if(!components.has(root)){
		result.set(root, amount);
		return result;
	}

	// add the root and amount to the result
	if(result.has(root)){
		result.set(root, result.get(root)! + amount);
	} else {
		result.set(root, amount);
	}

	// If the root is in the components map, we need to recursively call this function on all sub components and multiply the result by the amount
	const rootComponents = components.get(root)!; // we know this exists because root is calculated from this map
	for(const [name, subAmount] of rootComponents){
		const subResults = accumulateComponents(components, name, subAmount * amount);

		// combine sub results with result
		for(const [subName, subAmount] of subResults){
			if(result.has(subName)){
				result.set(subName, result.get(subName)! + subAmount);
			} else {
				result.set(subName, subAmount);
			}
		}
	}
	
	return result;
}

let svgcontainer: HTMLDivElement;

function calculateStructureGraph(components: Map<string, Map<string, number>>, root: string | null) {
	let result = "";

	result += "digraph G {\n";

	// make graph top to bottom
	result += "rankdir=TB;\n";

	// add subgraphs for each component
	for(const [name, parts] of components){
		result += "subgraph cluster_" + name + " {\n";
		result += "label = \"" + name + "\";\n";
		result += "color = blue;\n";
		result += "style = dashed;\n";
		for(const [part, amount] of parts){
			result += name + " -> " + part + " [label=\"" + amount + "\"];\n";
		}
		result += "}\n";
	}

	result += "}";
	return result;
}


function updateBoms(ev: any){
	components = parseComponents(bomtext);

	if(components.size === 0){
		return;
	}

	root = findRoot(components);

	componentsCumulative = accumulateComponents(components, root, 1);

	console.log(components);


	const structureGraph = calculateStructureGraph(components, root);

	instance().then(viz => {
		const svgresult = viz.renderSVGElement(structureGraph);
		svgcontainer.innerHTML = svgresult.outerHTML;
	});
}

</script>


<main>
	<nav class="bg-nav">
		<button on:click={goBack}>Zurück</button>
		<h1 class="flex-grow text-center">Stücklisten</h1>
	</nav>
	<div class="pa-30 flex-col">
		<div>
			<h1>Eingaben</h1>
			<p>
				Ins unten stehende Feld bitte die Materialstruktur eingeben. Jede Zeile definiert ein Material. Sie beginnt mit dem Materialnamen, dann Doppelpunkt. Alles danach definiert die Bestandteile des Materials vor dem Doppelpunkt. Nach dem Doppelpunkt folgen immer eine Anzahl und ein weiteres Material. Weitere Materialien müssen mit einem Komma abgetrennt werden. Beispiel <span class="example">A: 4 B, 5 C, 27 D</span>. Diese Zeile definiert das Material A, das aus 4 mal Material B, 5 mal Material C und 27 mal Material D besteht. Weitere Materialien können in einer neuen Zeile definiert werden.
			</p>
			<p>
				Die Namen der Materialien dürfen keine Leerzeichen enthalten. Menge und Material müssen durch ein Leerzeichen getrennt sein. Die Menge muss eine ganze Zahl sein. Falls mehr als 1 Hauptmaterial benötigt wird, kann man einfach ein Ersatzmaterial z.B. '_' als Hauptmaterial wählen, dass aus der gewünschten Anzahl des eigentlichen Hauptmaterials besteht. In der Dispotabelle muss der letzte Zeitabschnitt dann ignoriert werden.
			</p>
			<p>
				Das vorausgefüllte Beispiel entspricht genau dem Beispiel aus den Vorlesungsfolien.
			</p>
			{#if errmsg != null}
				<p class="errmsg">{errmsg}</p>
			{/if}
			<textarea class="bomtext" bind:value={bomtext} />
			<button on:click={updateBoms} class="bg-primary-btn text-white">Stücklisten erzeugen</button>
		</div>
		<div>
			<h1>Ergebnisse</h1>
			<span>Erkanntes Hauptmaterial: {root ? root : "Kein Hauptmaterial gefunden"}</span>
			<div>
				<h3>Mengenübersichtsstückliste</h3>
				<div>
					<table class="border-collapse text-left border-black">
						<tr class="bg-dark-gray">
							<th class="border-black text-center pa-15">Material</th>
							<th class="border-black pa-15">Menge</th>
						</tr>
						{#each componentsCumulative as comp}
							<tr class="border-black">
								<td class="text-center">{comp[0]}</td>
								<td class="text-center">{comp[1]}</td>
							</tr>
						{/each}
					</table>
				</div>
			</div>
			<div>
				<Dispotable components={components} root={root}/>
				<!--<h3>Dispositionstabelle</h3>
				<div>
					<table class="border-collapse text-left border-black">
						<tr>
							<th class="border-black pa-15 bg-dark-gray">Material</th>
							{#each Array(numTimeslots) as _, index}
								<th class="border-black pa-15">Zeit {index + 1}</th>
							{/each}
						</tr>
						{#each dispotable as dispitem}
							<tr class="border-black">
								<th class="text-center">{dispitem[0]}</th>
								{#each dispitem[1] as amount}
									<td class="text-center">{amount}</td>
								{/each}
							</tr>
						{/each}
					</table>
				</div>-->
			</div>
			<div>
				<h3>Baukastenstückliste</h3>
				<div class="flex-row flex-wrap g-15">
					{#each components as comp}
						<table class="border-collapse">
							<tr class="bg-dark-grey border-black text-left">
								<th colspan="3" class="ph-15 pv-5">{comp[0]} {comp[0] === root ? "(Hauptmaterial)" : ""}</th>
							</tr>
							<tr class="bg-dark-grey text-center">
								<th class="border-black pa-5">Pos</th>
								<th class="border-black pa-5">Ident-Nr</th>
								<th class="border-black pa-5">Menge</th>
							</tr>
							{#each comp[1] as part, index}
								<tr class="text-center">
									<td class="border-black pa-5">{index + 1}</td>
									<td class="border-black pa-5">{part[0]}</td>
									<td class="border-black pa-5">{part[1]}</td>
								</tr>
							{/each}
						</table>
					{/each}
				</div>
			</div>
			<div>
				<h3>Strukturstückliste</h3>
				<p>Todo... ist aber fast das gleiche was man eh eingeben muss...</p>
			</div>
			<div>
				<h3>Erzeugnisstruktur Baum (derzeit leider noch bisschen vermurkst)</h3>
				<div bind:this={svgcontainer} id="svgcontainer"></div>
			</div>
		</div>
	</div>
</main>

<style>
main{
	display: grid;
	grid-template-rows: 80px 1fr;
}

nav{
	display: flex;
	flex-direction: row;
	align-items: center;
	justify-content: space-between;
	padding: 1rem;
	box-shadow: 0px 0px 4px #8e8e8e;
}

.bomtext{
	width: 100%;
	height: 200px;
}

.example {
	background-color: #eaeaea;
	padding: 0.2rem;
	border-radius: 0.2rem;
	font: monospace;
}
</style>