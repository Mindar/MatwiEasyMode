<script lang="ts">
    import { onMount } from "svelte";


export let root: string | null;
export let components: Map<string, Map<string, number>>;

let materialsToProduce: {[key: string]: number} = {};

// default demand starts with 1 of the root material
let demand: number[] = [1];

onMount(() => {
	updateResults();
});

function removeDemand(index: number){
	demand.splice(index, 1);
	demand = demand;
	updateResults();
}

let lastDemand: string = "";
function addDemand(ev: any){
	const value = parseInt(lastDemand);

	if(isNaN(value)){
		return;
	}

	lastDemand = "";
	demand.push(value);

	demand = demand;


	updateResults();
}

type ProductionInfo = {
	time: number;
	materials: {[key: string]: number};
};

let productionTable: ProductionInfo[]= [];

function produceMaterials(time: number){
	// list of items to produce in next step
	const nextMaterialsToProduce: {[key: string]: number} = {};
	

	// Find production info for the current time
	let productionInfo = productionTable.find(p => p.time === time);

	if(productionInfo=== undefined) {
		productionInfo = {
			time: time,
			materials: {}
		};
		productionTable.push(productionInfo);
	}

	for(const mat in materialsToProduce) {
		// produce the material itself
		const amount = materialsToProduce[mat];

		// create or add to the produced material entry
		const matEntry = productionInfo.materials[mat];
		if(matEntry === undefined){
			productionInfo.materials[mat] = amount;
		} else {
			productionInfo.materials[mat] += amount;
		}

		const submatInfo = components.get(mat)!;

		if(submatInfo === undefined){
			// no submaterials, nothing to do
			continue;
		}
		console.log("Submaterials: ", submatInfo);

		// all submaterials of the material must be produced in the next time slot
		for(const [submat, submatsPerMat] of submatInfo){
			console.log(`Producing submat of ${mat}: ${submat} = ${submatsPerMat}`);
			nextMaterialsToProduce[submat] = (nextMaterialsToProduce[submat] ?? 0) + submatsPerMat * amount;
		}
	}

	// all materials in current step have been produced, next materials are now current materials
	materialsToProduce = nextMaterialsToProduce;
}



let toposortedMaterials: string[] = topoishsort(components, root);
function topoishsort(components: Map<string, Map<string, number>>, root: string | null): string[]{
	if(root === null || components.size === 0){
		return [];
	}

	const sorted: string[] = [];
	const visited: Set<string> = new Set<string>();
	
	const nextmats = [];
	nextmats.push(root);

	// return submats from the first element in nextmats until nextmats is empty
	while(nextmats.length > 0){
		const mat = nextmats.shift()!;

		if(visited.has(mat)){
			continue;
		} 
		visited.add(mat);
		sorted.push(mat);

		// add submats to nextmats
		const submats = components.get(mat);
		if(submats !== undefined){
			for(const submat of submats.keys()){
				nextmats.push(submat);
			}
		}
	}

	console.log("sorted mats toposr: ", sorted);
	return sorted;

}

let materialSet: Set<string> = new Set<string>();

function updateResults(){
	productionTable = [];


	console.log(demand);
	if(root === null || demand.length === 0 || components.size === 0){
		return;
	}

	let time = 0;

	const demandCopy = [...demand].reverse();

	// kickstart production with first material
	materialsToProduce[root] = demandCopy.shift()!;

	// while there are materials left to produce, produce them
	while(Object.keys(materialsToProduce).length > 0){
		console.group(`Run number ${time}, materials to produce: ${JSON.stringify(materialsToProduce)}`);
		produceMaterials(time++);

		// add demand for next cycle
		if(demandCopy.length > 0) {
			materialsToProduce[root] = (materialsToProduce[root] ?? 0) + demandCopy.shift()!;
		}
		console.log(productionTable);
		console.groupEnd();
	}

	materialSet = new Set<string>();
	for(const prod of productionTable){
		for(const mat in prod.materials){
			materialSet.add(mat);
		}
	}

	// all materials have been produced
	// now we must sort the production table by time
	productionTable.sort((a, b) => b.time - a.time);

	productionTable = productionTable;
}

</script>

<div>
	<h3>Dispositionstabelle</h3>
	<div>
		<h4>Bedarfe für Material {root}</h4>
		<div class="flex-col g-5">
			{#each demand as d, index(index)}
				<div class="flex-row">
					<input bind:value={demand[index]} type="number" on:change={() => {demand = demand; updateResults();}} />
					<button on:click={() => removeDemand(index)} class="bg-danger-btn text-white pa-5">x</button>
				</div>
			{/each}
			<div class="flex-row">
				<input type="number" bind:value={lastDemand} on:change={addDemand}/>
				<button on:click={addDemand} class="bg-primary-btn text-white pa-5">+</button>
			</div>
		</div>
	</div>
	<div>
		<h4>Disposition</h4>
		<div>
			<table class="border-collapse border-black">
				<tr class="bg-dark-grey border-black">
					<th class="pa-5">Zeit</th>
					<!-- Column for each timeslot in productionTable -->
					{#each productionTable as prodInfo, index}
						<th class="border-black pa-5">{productionTable.length - prodInfo.time}</th>
					{/each}
				</tr>
				{#each toposortedMaterials as mat}
					<tr>
						<th class="border-black">{mat}</th>
						{#each productionTable as prodInfo}
							<td class="border-black text-right pa-5" class:highlightCell={prodInfo.materials[mat] != undefined && prodInfo.materials[mat] !== 0}>{(prodInfo.materials[mat] != undefined) && (prodInfo.materials[mat] !== 0) ? prodInfo.materials[mat] : ""}</td>
						{/each}
					</tr>
				{/each}
			</table>
		</div>
	</div>
</div>


<style>
.highlightCell {
	/* slight green tint*/
	background-color: #caffca;
}
</style>