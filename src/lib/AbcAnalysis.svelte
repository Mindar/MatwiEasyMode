<script lang="ts">
    import { createEventDispatcher } from "svelte";


const dispatch = createEventDispatcher();

function goBack() {
	dispatch("goBack");
}

let aLimit = 80;
let bLimit = 15;



let items = [
	{
		id: Math.floor(Math.random() * 1000),
		name: "Beispiel",
		amount: 100,
		price: 10
	}
]

function addArticle(){
	const id = Math.floor(Math.random() * 1000);
	items.push({
		id,
		name: "Beispiel" + id,
		amount: 0,
		price: 0
	});

	items = items;

	updateResults();
}

function removeItem(item: any){
	items.splice(items.indexOf(item), 1);
	items = items;

	updateResults();
}

let aItems: any[] = [];
let bItems: any[] = [];
let cItems: any[] = [];

let aTotal = 0;
let bTotal = 0;
let cTotal = 0;

let total = 0;


// format numbers with 2 decimals max
const fmt = new Intl.NumberFormat("de-DE", {
	minimumFractionDigits: 2,
	maximumFractionDigits: 2
});


function updateResults(){
	console.log("items usnorted:", items);

	// copy the items and sort the list by total price
	const sortedItems = [...items].sort((a, b) => {
		return (b.amount * b.price) - (a.amount * a.price);
	});
	console.log("items soretd:", sortedItems);

	// clear the lists
	aItems = [];
	bItems = [];
	cItems = [];

	total = sortedItems.reduce((acc, item) => {
		return acc + item.amount * item.price;
	}, 0);

	aTotal = 0;
	bTotal = 0;
	cTotal = 0;

	for(const item of sortedItems) {
		// we iterate the list and add items to the list of A items until the A-items total value exceeds the a limit, then we do the same for B and C
		if(aTotal/total < aLimit/100){
			aTotal += item.amount * item.price;
			aItems.push(item);
		} else {
			if(bTotal/total < bLimit/100){
				bTotal += item.amount * item.price;
				bItems.push(item);
			} else {
				cTotal += item.amount * item.price;
				cItems.push(item);
			}
		}
	}

	aItems = aItems;
	bItems = bItems;
	cItems = cItems;
}

</script>


<main>
	<nav class="bg-nav">
		<button on:click={goBack}>Zurück</button>
		<h1 class="flex-grow text-center">ABC Analyse</h1>
	</nav>
	<div class="pa-30 flex-col">
		<div class="flex-row space-around">
			<label>
				A-Grenze
				<input type="number" min="0" max="100" bind:value={aLimit} on:input={updateResults}/>
			</label>
			<label>
				B-Grenze
				<input type="number" min="0" max="100" bind:value={bLimit} on:input={updateResults}/>
			</label>
			<label>
				C-Grenze
				<input type="number" min="0" max="100" value={100 - aLimit - bLimit} disabled/>
			</label>
		</div>

		<div class="flex-col pv-30">
			<div class="flex-col">
				<h2>Artikelliste</h2>


				{#each items as item(item.id)}
					<div class="flex-row align-center g-15 mb-30">
						<label>
							Artikel
							<input type="text" bind:value={item.name} on:input={updateResults} />
						</label>
						<label>
							Menge
							<input type="number" bind:value={item.amount} on:input={updateResults} />
						</label>
						<label>
							Einzelpreis €
							<input type="number" bind:value={item.price} on:input={updateResults} />
						</label>
						<button class="bg-danger-btn text-white pa-5" on:click={() => removeItem(item)}>X</button>
					</div>
				{/each}
				<button class="fsize-2em pa-5 bg-primary-btn text-white bold" on:click={addArticle}>+</button>
			</div>


			<div class="flex-col pv-30">
				<h2>Ergebnis</h2>
				<table>
					<tr>
						<th>Artikel</th>
						<th>Wert</th>
					</tr>
					{#each aItems as item(item.id)}
						<tr class="aItem">
							<td>{item.name}</td>
							<td>{item.amount * item.price}</td>
							<td>A-Teil</td>
						</tr>
					{/each}
					<tr class="aItem">
						<th>A-Summe</th>
						<th>{aTotal}</th>
						<th>{fmt.format(aTotal/total * 100)} %</th>
					</tr>
					{#each bItems as item(item.id)}
						<tr class="bItem">
							<td>{item.name}</td>
							<td>{item.amount * item.price}</td>
							<td>B-Teil</td>
						</tr>
					{/each}
					<tr class="bItem">
						<th>B-Summe</th>
						<th>{bTotal}</th>
						<th>{fmt.format(bTotal/total * 100)} %</th>
					</tr>
					{#each cItems as item(item.id)}
						<tr class="cItem">
							<td>{item.name}</td>
							<td>{item.amount * item.price}</td>
							<td>C-Teil</td>
						</tr>
					{/each}
					<tr class="cItem">
						<th>C-Summe</th>
						<th>{cTotal}</th>
						<th>{fmt.format(cTotal/total * 100)} %</th>
					</tr>
					<tr>
						<td class="text-center">Gesamt</td>
						<td class="text-center">{total}</td>
					</tr>
				</table>
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

.aItem{
	background-color: #8ef08e;
}

.bItem{
	background-color: #f8f898;
}

.cItem{
	background-color: #fcaeae;
}

table {
	border-collapse: collapse;
}

table td, table th {
	padding: 0.5rem;
}
</style>