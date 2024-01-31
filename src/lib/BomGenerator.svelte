<script lang="ts">
    import { createEventDispatcher } from "svelte";


const dispatch = createEventDispatcher();

function goBack() {
	dispatch("goBack");
}


</script>


<main>
	<nav class="bg-nav">
		<button on:click={goBack}>Zurück</button>
		<h1 class="flex-grow text-center">Stücklisten</h1>
	</nav>
	<div>
		todo: aus strukturstüli Mengenstückliste generieren, dispo listengedöns generieren, evtl noch input für lagerstände
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