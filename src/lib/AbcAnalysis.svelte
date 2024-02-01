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
		id: Math.floor(Math.random() * 1000000),
		name: "Beispiel",
		amount: 100,
		price: 10
	}
]

function addArticle(){
	// if the previous article had an integer number as name, increment the name by one and add the new article
	const lastItem = items[items.length - 1];
	const nameMatches = lastItem.name.match(/^[^\d]*(\d+)$/);
	
	// if name matched the pattern, increment the number and add the new article
	if(nameMatches){
		console.log(nameMatches);
		console.log(nameMatches[0]);
		const number = parseInt(nameMatches[1]);
		const name = lastItem.name.replace(/\d+$/, "") + (number + 1);
		items.push({
			id: Math.floor(Math.random() * 1000000),
			name,
			amount: 1,
			price: 0
		});
	} else {
		// otherwise just add a new article with the default name
		items.push({
			id: Math.floor(Math.random() * 1000000),
			name: "Beispiel",
			amount: 1,
			price: 0
		});
	}

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
let consumptionTotal = 0;


let aItemsConsumptionTotal = 0;
let aItemsConsumptionRel = 0;
let bItemsConsumptionTotal = 0;
let bItemsConsumptionRel = 0;
let cItemsConsumptionTotal = 0;
let cItemsConsumptionRel = 0;

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

	let accumulator = 0;

	let itemClass = "a";

	consumptionTotal = items.reduce((acc, item) => {
		return acc + item.amount;
	}, 0);

	for(const item of sortedItems) {
		const itemValue = item.amount * item.price;
		// accumulate relative percentages of item values
		accumulator += itemValue/total * 100;

		const consumptionRel = item.amount / consumptionTotal;


		switch(itemClass) {
			case "a":
				if(((aTotal + itemValue)/total) > (aLimit/100)){
					itemClass = "b";
					bTotal += itemValue;
					bItems.push({
						...item,
						accumulator,
						consumptionRel
					});
				} else {
					aTotal += itemValue;
					aItems.push({
						...item,
						accumulator,
						consumptionRel
					});
				}
				break;
			case "b":
				if(((aTotal + bTotal + itemValue)/total) > ((aLimit + bLimit)/100)){
					itemClass = "c";
					cTotal += itemValue;
					cItems.push({
						...item,
						accumulator,
						consumptionRel
					});
				} else {
					bTotal += itemValue;
					bItems.push({
						...item,
						accumulator,
						consumptionRel
					});
				}
				break;
			case "c":
				cTotal += itemValue;
				cItems.push({
					...item,
					accumulator,
					consumptionRel
				});
				break;
		}
	}


	aItemsConsumptionTotal = aItems.reduce((acc, item) => {
		return acc + item.amount;
	}, 0);
	aItemsConsumptionRel = aItems.reduce((acc, item) => {
		return acc + item.consumptionRel;
	}, 0);
	bItemsConsumptionTotal = bItems.reduce((acc, item) => {
		return acc + item.amount;
	}, 0);
	bItemsConsumptionRel = bItems.reduce((acc, item) => {
		return acc + item.consumptionRel;
	}, 0);
	cItemsConsumptionTotal = cItems.reduce((acc, item) => {
		return acc + item.amount;
	}, 0);
	cItemsConsumptionRel = cItems.reduce((acc, item) => {
		return acc + item.consumptionRel;
	}, 0);

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
				<table class="text-center">
					<tr>
						<th>Artikel</th>
						<th>Verbrauch [Stk]</th>
						<th>Preis [€]</th>
						<th>Verbrauchswert [€]</th>
						<th>Verbrauchswert [%]</th>
						<th>Verbrauchswert (kumuliert) [%]</th>
						<th>Verbrauch [%]</th>
					</tr>
					{#each aItems as item(item.id)}
						<tr class="aItem">
							<td>{item.name}</td>
							<td>{item.amount}</td>
							<td>{item.price}</td>
							<td>{item.amount * item.price}</td>
							<td>{fmt.format(100 * item.amount * item.price / total)} %</td>
							<td>{fmt.format(item.accumulator)} %</td>
							<td>{fmt.format(100 * item.consumptionRel)} %</td>
						</tr>
					{/each}
					<tr class="aItem">
						<th>A-Summe</th>
						<th>{aItemsConsumptionTotal}</th>
						<th></th>
						<th>{aTotal}</th>
						<th>{fmt.format(aTotal/total * 100)} %</th>
						<th>{fmt.format(aTotal/total * 100)} %</th>
						<th>{fmt.format(100 * aItemsConsumptionRel)} %</th>
					</tr>
					{#each bItems as item(item.id)}
						<tr class="bItem">
							<td>{item.name}</td>
							<td>{item.amount}</td>
							<td>{item.price}</td>
							<td>{item.amount * item.price}</td>
							<td>{fmt.format(100 * item.amount * item.price / total)} %</td>
							<td>{fmt.format(item.accumulator)} %</td>
							<td>{fmt.format(100 * item.consumptionRel)} %</td>
						</tr>
					{/each}
					<tr class="bItem">
						<th>B-Summe</th>
						<th>{bItemsConsumptionTotal}</th>
						<th></th>
						<th>{bTotal}</th>
						<th>{fmt.format(bTotal/total * 100)} %</th>
						<th>{fmt.format((aTotal + bTotal)/total * 100)} %</th>
						<th>{fmt.format(100 * bItemsConsumptionRel)} %</th>
					</tr>
					{#each cItems as item(item.id)}
						<tr class="cItem">
							<td>{item.name}</td>
							<td>{item.amount}</td>
							<td>{item.price}</td>
							<td>{item.amount * item.price}</td>
							<td>{fmt.format(100 * item.amount * item.price / total)} %</td>
							<td>{fmt.format(item.accumulator)} %</td>
							<td>{fmt.format(100 * item.consumptionRel)} %</td>
						</tr>
					{/each}
					<tr class="cItem">
						<th>C-Summe</th>
						<th>{cItemsConsumptionTotal}</th>
						<th></th>
						<th>{cTotal}</th>
						<th>{fmt.format(cTotal/total * 100)} %</th>
						<th>{fmt.format((aTotal + bTotal + cTotal)/total * 100)} %</th>
						<th>{fmt.format(100 * cItemsConsumptionRel)} %</th>
					</tr>
					<tr>
						<td class="text-center">Gesamt</td>
						<td class="text-center">{consumptionTotal}</td>
						<td class="text-center"></td>
						<td class="text-center">{total}</td>
					</tr>
				</table>
			</div>
			<div class="info">
				<h4>
					Hinweise
				</h4>
				<p>
					Beim Hinzufügen neuer Artikel ist die voreingestellte Menge immer 1. Wenn man also den Gesamtwert schon vorgegeben hat, kann man also auch einfach diesen als "Einzelpreis" eingeben und spart sich somit Tippaufwand.
				</p>
				<p>
					Wenn Materialnummern als Name genutzt werden, dann wird die Nummer bei Klick auf "+" immer um eins erhöht. Es macht also Sinn mit der niedrigsten Materialnummer anzufangen und dann die höheren einzugeben. Das funktioniert übrigens auch mit Lücken zwischen den Nummern. Maßgeblich ist dann immer die letzte hinzugefügte Materialnummer.
				</p>
				<p>
					<strong>Achtung:</strong> Bitte unbedingt nochmal die Klassifizierung der Artikel überprüfen. Unser Prof war in seinen eigenen Unterlagen nicht immer konsistent was die Zuordnung zu den Klassen A, B und C angeht. Derzeit ist es so, dass Artikel der nächsten Klasse zugeordnet werden, wenn die Grenze durch den Artikel überschritten wäre. Das heißt konkret, wenn die Grenze von A bei 85% liegt, der aktuelle Wert bei 80% des Gesamtwerts liegt und der nächste Artikel einen Wertanteil von 6% hätte, dann würde dieser Artikel nicht mehr der Klasse A, sondern der Klasse B zugeordnet, da Klasse A mit 86% sonst über der Grenze läge. Der Anteil von A bleibt damit bei 80%, die Grenze 85% wird also nie überschritten. Das war in den Unterlagen meistens auch so und ist vermutlich richtig. In den Unterlagen gab es aber auch Einzelfälle bei denen es nicht so war (lag vermutlich in diesen Fällen an Rundungsfehlern, aber wer weiß...).
				</p>
				<p>
					Am besten einfach nur die Werte nutzen und die Klassen selbst bestimmen. Das ist ja dann glücklicherweise easy wenn man die Werte hat.
				</p>
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

.info p {
	margin-top: 1em;
	padding: 0px;
}
</style>