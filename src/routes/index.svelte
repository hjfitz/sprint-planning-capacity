<template>
	<header>
		<ul class="grid grid-cols-5 my-2 gap-2">
			<li class="text-xl">Developer</li>
			<li class="text-xl">Days Spent Working</li>
			<li class="text-xl">Days Holiday</li>
			<li class="text-xl">Days With Other Commitments</li>
			<li class="text-xl">Days for Development</li>
		</ul>
	</header>
	<ul>
		{#each developers as dev, i}
		<li class="grid grid-cols-5 gap-2 my-2">
			<input bind:value={dev.name}/>
			<input type=number bind:value={dev.daysWorking} on:input={() => recalculateDevDays(i)} />
			<input type=number bind:value={dev.daysHoliday} on:input={() => recalculateDevDays(i)} />
			<input type=number bind:value={dev.daysOtherCommitment} on:input={() => recalculateDevDays(i)} />
			<p> {dev.devDays} </p>
		</li>
		{/each}
	</ul>

	<p>Total dev days: {totalDevDays}</p>
	<p>Total with buffer: {totalWithBuffer}</p>
	<p>Are we on support? <button on:click={toggleSupport}>{support ? 'yes' : 'no'}</button></p>
	<p class={support ? '' : 'hidden'}>Total with support buffer: {totalWithSupportBuffer}</p> 
	<p>Average points per dev day: <input type=number bind:value={avgPoints} /></p>
	<p>Guideline sprint points: {avgPoints * totalWithSupportBuffer}</p>

	<button on:click={addDeveloper}>Add Developer</button>
	<button on:click={removeDeveloper}>Remove Developer</button>

</template>
<script lang="ts">
	interface Developer {
		name: string
		daysHoliday: number
		daysOtherCommitment: number
		daysWorking: number
		devDays: number
		notes?: string
	}
	
	const DEV_STORE_KEY = 'dev-cache'

	function isWindowUndefined(): boolean {
		return typeof window === 'undefined'
	}

	function recalculateDevDays(idx: number) {
		const dev = developers[idx]
		dev.devDays = dev.daysWorking - (dev.daysOtherCommitment + dev.daysHoliday)
		// needed for reactivity
		developers = developers
	}

	function developerFactory(): Developer {
		return {
			name: 'Developer',
			daysHoliday: 0,
			daysOtherCommitment: 0,
			daysWorking: 8,
			notes: undefined,
			devDays: 8
		}
	}

	function toggleSupport() {
		support = !support
	}

	function addDeveloper() {
		developers = [...developers, developerFactory()]
	}

	function removeDeveloper() {
		developers.pop()
		developers = developers
	}

	// todo: include on call/release numbers and dev points per day
	function updateDevelopers() {
		if (isWindowUndefined()) return
		const stringified = JSON.stringify(developers)
		localStorage.setItem(DEV_STORE_KEY, stringified)
	}

	function getDevelopers() {
		const defaultDevelopers = Array.from({length: 5}, developerFactory)
		if (isWindowUndefined()) return defaultDevelopers
		const stored = localStorage.getItem(DEV_STORE_KEY)
		if (stored) {
			return JSON.parse(stored) as Developer[]
		} else {
			return defaultDevelopers
		}
	}

	function formatDecimal(num: number): number {
		return +num.toFixed(2)
	}

	let avgPoints = 1
	let support = false
	let developers = getDevelopers()

	$: totalDevDays = developers.map(dev => dev.devDays).reduce((acc, cur) => acc + cur, 0)
	$: totalWithBuffer = formatDecimal(totalDevDays * 0.85)
	$: totalWithSupportBuffer = support ? formatDecimal(totalWithBuffer * 0.95) : totalWithBuffer
	$: developers, updateDevelopers()

</script>
