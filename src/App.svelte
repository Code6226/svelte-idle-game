<script>
	import { onMount, onDestroy } from "svelte";
	import _ from "lodash"

	console.log('init')
	// export let name;
	let amount = 0
	let perSecond = 0
	let costAddPerSecond = 2

	let previousTime = Date.now()
	let interval;

	// ~Computed
	$: addPerClick = Math.max(Math.floor(perSecond / 2), 1);
	$: addPerSecond = Math.max(Math.floor(addPerClick / 2), 1);

	// ~Observer/Watch, which triggers if any of the used vars have changed
	$: {
		console.log(`The addPerClick has changed to: ${addPerClick}`);
	}

	onMount(function (){
		console.log('onMount')
		loadState()
		interval = setInterval(function (){
			let now = Date.now()
			let secElapsed = Math.round((now - previousTime) / 1000)
			previousTime = now
			console.log('secElapsed', secElapsed)

			amount += perSecond * secElapsed
			saveState()
		}, 1000)
	})

	onDestroy(function (){
		clearInterval(interval)
	})

	function loadState(){
		try{
			let stored = JSON.parse(localStorage.getItem('myStuff'))
			amount = stored.amount
			perSecond = stored.perSecond
			costAddPerSecond = stored.costAddPerSecond || 2
			previousTime = Number.isFinite(previousTime) ? stored.previousTime : Date.now()
		}catch (e) {
			console.log('Could not load state')
		}
	}
	function saveState(){
		localStorage.setItem('myStuff', JSON.stringify({amount, perSecond, costAddPerSecond, previousTime}))
	}

	function onClickAddOnce(){
		amount += addPerClick
	}
	function onClickAddPerSec(){
		if(amount >= costAddPerSecond){
			amount -= costAddPerSecond
			perSecond += addPerSecond
			costAddPerSecond *= 2
		}
	}
</script>

<main>
	<h1>Amount: {amount}</h1>
	<h3>+ {perSecond}/s</h3>

	<div>
		<button on:click={onClickAddOnce}>+ {addPerClick}</button>
	</div>
	<div>
		<button on:click={onClickAddPerSec} disabled="{costAddPerSecond > amount}">Buy + {addPerSecond}/s for {costAddPerSecond}</button>
	</div>

	{#if amount === 11}
		Eleven!
	{:else}
		<!-- maybe later -->
		{#each _.range(3) as addPerSecond, i}
			<span class="not-eleven-{i}"/>
		{/each}
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>