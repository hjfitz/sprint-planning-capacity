<template>
	<main class="grid auto-rows-min content-between min-h-full ">
		<section class="container mx-auto">
			<div class="overflow-y-auto max-w-[100vw]">
				<table class="mt-4 border rounded-md shadow w-full min-w-[640px]">
					<thead>
						<tr class="grid grid-cols-5 py-4 gap-2 border-b bg-gray-100">
							<th class="font-semibold">Developer</th>
							<th class="font-semibold">Days Spent Working</th>
							<th class="font-semibold">Days Holiday</th>
							<th class="font-semibold">Days With Other Commitments</th>
							<th class="font-semibold">Days for Development</th>
						</tr>
					</thead>

					<tbody>
						{#each developers as dev, i}
						<tr class="grid grid-cols-5 gap-2 p-2 border-b">
							<td><input class="max-w-full font-semibold p-2 border-white border-2 hover:border-gray-200" bind:value={dev.name}/></td>
							<td><input class="max-w-full p-2 border-white border-2 hover:border-gray-200" type=number bind:value={dev.daysWorking} on:input={recalculateDevDaysFactory(i)} /></td>
							<td><input class="max-w-full p-2 border-white border-2 hover:border-gray-200" type=number bind:value={dev.daysHoliday} on:input={recalculateDevDaysFactory(i)} /></td>
							<td><input class="max-w-full p-2 border-white border-2 hover:border-gray-200" type=number bind:value={dev.daysOtherCommitment} on:input={recalculateDevDaysFactory(i)} /></td>
							<td><p class="p-2 border-white border-2 hover:border-gray-200 cursor-not-allowed">{dev.devDays}</p></td>
						</tr>
						{/each}
					</tbody>
				</table>
			</div>

			<section class="pt-8 container mx-auto">
				<h1 class="text-2xl">Outcomes:</h1>
				<div class="grid grid-cols-1 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 py-4 gap-8">
					<InfoItem text="Total dev days" icon="📆">{totalDevDays}</InfoItem>
					<InfoItem text="Total with buffer" icon="🗓️"> {totalWithBuffer}</InfoItem>
					<InfoItem text="Are we on support?" icon="🔥">
						<button class="bg-gray-200 mt-2 px-2 py-1 rounded-md shadow w-3/5 hover:bg-gray-100 transition duration-300" on:click={toggleSupport}>{support ? 'yes' : 'no'}</button>
					</InfoItem>
					<InfoItem className={support ? '' : 'hidden'} text="Total with support buffer" icon="🧰">{totalWithSupportBuffer}</InfoItem> 
					<InfoItem text="Avg points per dev day" icon="📲">
						<input class="border rounded px-2 py-1 mt-2 w-4/5" type=number bind:value={avgPoints} />
					</InfoItem>
					<InfoItem text="Guideline sprint points" icon="💻"><p class="inline" on:click={copyPoints}>{formatDecimal(avgPoints * totalWithSupportBuffer)}</p></InfoItem>
				</div>
			</section>
		</section>

		<section class="self-end flex justify-around sm:flex-row items-center flex-col py-8 bg-gray-100">
			<button class="hover:bg-gray-300 bg-gray-200 my-2 w-64 shadow transition duration-300 py-2 px-8 rounded" on:click={addDeveloper}>Add Developer</button>
			<button class="hover:bg-gray-300 bg-gray-200 my-2 w-64 shadow transition duration-300 py-2 px-8 rounded" on:click={removeDeveloper}>Remove Developer</button>
		</section>
	</main>
</template>
<script lang="ts">
	import InfoItem from '../components/InfoItem.svelte';

	interface Developer {
		name: string
		daysHoliday: number
		daysOtherCommitment: number
		daysWorking: number
		devDays: number
		notes?: string
	}
	

	function getTotalPoints(): number {
		return avgPoints * totalWithSupportBuffer
	}

	function copyPoints() {
		return navigator.clipboard.writeText(getTotalPoints().toString())
	}

	function isWindowUndefined(): boolean {
		return typeof window === 'undefined'
	}

	function recalculateDevDaysFactory(idx: number) {
		return function recalculateDevDays() {
			const dev = developers[idx]
			dev.devDays = dev.daysWorking - (dev.daysOtherCommitment + dev.daysHoliday)
			// needed for reactivity
			developers = developers
		}
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



	function formatDecimal(num: number): number {
		return +num.toFixed(2)
	}

	const DEV_STORE_KEY = 'dev-cache'
	const POINTS_CACHE_KEY = 'points-cache'
	const SUPPORT_CACHE_KEY = 'support-cache'

	// todo: gets and updates are very WET - we could probably simplify this.
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

	function getAveragePoints(): number {
		const DEFAULT_POINTS = 1
		if (isWindowUndefined()) return DEFAULT_POINTS
		// null cast to number is 0
		const points = +localStorage.getItem(POINTS_CACHE_KEY)
		if (!points) return DEFAULT_POINTS
		return points
	}

	function getOnSupport(): boolean {
		const DEFAULT_SUPPORT = false
		if (isWindowUndefined()) return DEFAULT_SUPPORT
		const onSupport = localStorage.getItem(SUPPORT_CACHE_KEY)
		if (!onSupport) return DEFAULT_SUPPORT
		return onSupport === 'true'
	}

	function updateDevelopers() {
		if (isWindowUndefined()) return
		const stringified = JSON.stringify(developers)
		localStorage.setItem(DEV_STORE_KEY, stringified)
	}

	function updateAvgPoints() {
		if (isWindowUndefined()) return
		localStorage.setItem(POINTS_CACHE_KEY, avgPoints?.toString())
	}

	function updateOnSupport() {
		if (isWindowUndefined()) return
		localStorage.setItem(SUPPORT_CACHE_KEY, support.toString())
	}

	let avgPoints = getAveragePoints()
	let support = getOnSupport()
	let developers = getDevelopers()

	$: totalDevDays = developers.map(dev => dev.devDays).reduce((acc, cur) => acc + cur, 0)
	$: totalWithBuffer = formatDecimal(totalDevDays * 0.85)
	$: totalWithSupportBuffer = support ? formatDecimal(totalWithBuffer * 0.95) : totalWithBuffer
	$: developers, updateDevelopers()
	$: avgPoints, updateAvgPoints()
	$: support, updateOnSupport()

</script>
