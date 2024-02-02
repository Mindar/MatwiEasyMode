<script lang="ts">
    import { createEventDispatcher } from "svelte";


const dispatch = createEventDispatcher();

function goBack() {
	dispatch("goBack");
}


let yearlyDemand = 5000;
let itemPrice = 100;
let orderCosts = 1;
let holdingCosts = 3;
let optimalOrderQuantity = 0;
let numOrders = 0;
let lastOrderQuantity = 0;


function andler(_: any){
	const numerator = 200 * yearlyDemand * orderCosts;
	const denominator = holdingCosts * itemPrice;

	optimalOrderQuantity = Math.sqrt(numerator / denominator);
	numOrders = yearlyDemand / optimalOrderQuantity;
	lastOrderQuantity = yearlyDemand - optimalOrderQuantity * numOrders;
}



let initialPrice = 100;
let materialCostRel = 50;
let materialCostInitial = 50;
let materialCost = 60;
let salaryCostRel = 50;
let salaryCostInitial = 50;
let salaryCost = 60;
</script>


<main>
	<nav class="bg-nav">
		<button on:click={goBack}>Zurück</button>
		<h1 class="flex-grow text-center">Sonstiges</h1>
	</nav>
	<div class="pa-30">
		<div>
			<h3>Losgrößenformel nach Andler</h3>
			<div class="flex-col g-5">
				<label>
					<span>Jährlicher Bedarf (JB) [Stk]:</span>
					<input type="number" bind:value={yearlyDemand} on:input={andler} />
				</label>
				<label>
					<span>Preis pro Stück (p) [€]:</span>
					<input type="number" bind:value={itemPrice} on:input={andler} />
				</label>
				<label>
					<span>Bestellkosten (Kb) [€]:</span>
					<input type="number" bind:value={orderCosts} on:input={andler} />
				</label>
				<label>
					<span>Lagerhaltungskostenfaktor (Lf) [%]:</span>
					<input type="number" bind:value={holdingCosts} on:input={andler} />
				</label>
				<span>
					Optimale Bestellmenge: {optimalOrderQuantity} Stk, Optimale Bestellanzahl: {numOrders} Bestellungen
				</span>
				<span>
					Beste Bestellstrategie: {Math.round(numOrders)} Bestellungen á {Math.round(optimalOrderQuantity)} Stk für {Math.round(numOrders) * Math.round(optimalOrderQuantity)} Stk
				</span>
				<span>
					Bestellüberschuss durch Rundungsfehler: {Math.round(optimalOrderQuantity) * Math.round(numOrders) - yearlyDemand} Stk
				</span>
			</div>
		</div>
		<div>
			<h3>Preiskopplung an Kostenelementklauseln</h3>
			<p>
				In den Folien wurde teilweise nicht der Anfangs und Endwert der Material- und Lohnkosten Angegeben, sondern nur die Entwicklung bzw eine prozentuale Erhöhung. In Solchen Fällen einfach 100 als Anfangswert eintragen und als Endwert dann 100 + Erhöhung in Prozent eintragen. Teilweise waren auch nur Werte wie z.B. Im=1,4 gegeben. In diesem Fall kann man 1 als Anfangswert und 1.4 als Endwert eintragen. (Beachten, dass ein PUNKT als Dezimaltrennzeichen genutzt werden muss.)
			</p>
			<div class="flex-col g-5">
				<label>
					<span>Anfangspreis (P0) [€]:</span>
					<input type="number" bind:value={initialPrice} on:input={andler} />
				</label>

				<label>
					<span>Materialkostenanteil (b) [%]:</span>
					<input type="number" bind:value={materialCostRel} on:input={andler} />
				</label>
				<label>
					<span>Materialkosten damals (M0) [€]:</span>
					<input type="number" bind:value={materialCostInitial} on:input={andler} />
				</label>
				<label>
					<span>Materialkosten heute (M1) [€]:</span>
					<input type="number" bind:value={materialCost} on:input={andler} />
				</label>
				
				<label>
					<span>Lohnkostenanteil (b) [%]:</span>
					<input type="number" bind:value={salaryCostRel} on:input={andler} />
				</label>
				<label>
					<span>Lohnkosten damals (L0) [€]:</span>
					<input type="number" bind:value={salaryCostInitial} on:input={andler} />
				</label>
				<label>
					<span>Lohnkosten heute (L1) [€]:</span>
					<input type="number" bind:value={salaryCost} on:input={andler} />
				</label>

				<span>
					Ermittelter Kosten-Restanteil: {100 - materialCostRel - salaryCostRel}% (darf nicht negativ sein)
				</span>
				<span>
					Angepasster Preis: {initialPrice * (materialCostRel / 100) * (materialCost / materialCostInitial) + initialPrice * (salaryCostRel / 100) * (salaryCost / salaryCostInitial)} €
				</span>
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

label{
	display: flex;
	flex-direction: row;
	align-items: center;
	justify-content: space-between;
}
</style>