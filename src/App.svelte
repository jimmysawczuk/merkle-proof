<script>
	import { csvParse } from "d3"
	import { ethers } from "ethers"
	import { MerkleTree } from "merkletreejs"
	import "buffer"

	const availableTypes = ["IGNORE", "address", "uint256", "uint64"]

	let rawCSV = ``

	let data = []
	let headers = []
	let types = {}

	let tree = null
	let legend = null
	let legendKey = null
	$: root = tree?.getHexRoot() ?? null
	$: leaves = tree?.getHexLeaves() ?? null

	async function parseCSV() {
		data = csvParse(rawCSV)
		headers = data.columns
		updateRoot()
	}

	function updateRoot() {
		const leaves = []
		const leg = []

		const ty = headers
			.map((header) => {
				return types[header]
			})
			.filter((v) => v !== "IGNORE")

		for (const row of data) {
			const r = headers
				.map((header) => {
					if (types[header] === "IGNORE") return null
					return row[header]
				})
				.filter((v) => v !== null)

			const leaf = ethers.utils.solidityKeccak256(ty, r)
			leaves.push(leaf)
			leg.push({
				...row,
				hash: leaf,
			})
		}

		tree = new MerkleTree(leaves, ethers.utils.keccak256, { sort: true })
		legend = buildLegend(leg)
	}

	function buildLegend(entries) {
		if (legendKey === null) {
			return entries
		}

		const tbr = {}
		for (const row of entries) {
			tbr[row[legendKey]] = row
		}
		return tbr
	}

	parseCSV()
</script>

<main class="w-screen h-screen grid grid-cols-2 relative">
	<div class="p-4 h-full max-h-screen grid grid-rows-[1fr,3fr]">
		<div>
			<p class="text-lg leading-[2rem]">Paste your CSV here:</p>
			<div class="h-48">
				<textarea
					class="w-full h-full resize-none block rounded-md text-sm font-mono p-2 text-slate-900 border border-slate-900/30 dark:border-slate-50/30 transition-colors"
					bind:value={rawCSV}
					on:keyup={parseCSV}
				/>
			</div>
		</div>

		<div class="mt-4 overflow-y-scroll rounded-md">
			{#if data.length > 0}
				<table class="relative w-full">
					<thead class="text-left sticky top-0">
						<tr class="bg-sky-800">
							{#each headers as header}
								<th>
									<div class="text-white px-2 pt-2">
										{header}
									</div>
									<div class="px-2 pb-2 mt-2">
										<select
											bind:value={types[header]}
											on:change={updateRoot}
											class="font-normal block w-full py-1 bg-slate-50 dark:bg-slate-900 transition-colors rounded-sm text-sm"
										>
											<option value={null} disabled
												>Select a type</option
											>
											{#each availableTypes as ty}
												<option value={ty}>{ty}</option>
											{/each}
										</select>
									</div>
								</th>
							{/each}
						</tr>
					</thead>
					<tbody>
						{#each data as row}
							<tr
								class="odd:bg-slate-100 even:bg-slate-200 dark:odd:bg-slate-700 dark:even:bg-slate-800 transition-colors"
							>
								{#each headers as header}
									<td class="px-2 py-1">{row[header]}</td>
								{/each}
							</tr>
						{/each}
					</tbody>
				</table>
			{/if}
		</div>
	</div>

	<div
		class="space-y-6 p-4 bg-slate-300 dark:bg-slate-600 transition-colors sticky top-0"
	>
		<div class="bg-sky-800 text-white p-4 rounded-md">
			<label class="block font-bold" for="legend-key-select">
				Legend key:
			</label>
			<select
				id="legend-key-select"
				bind:value={legendKey}
				on:change={updateRoot}
				class="text-base font-normal block w-full py-2 px-2 bg-slate-50 mt-2 rounded-md text-slate-900"
			>
				<option value={null}>No key</option>
				{#each headers as header}
					<option value={header}>{header}</option>
				{/each}
			</select>
		</div>

		<div class="overflow-x-hidden w-full">
			<div class="font-bold">Root</div>
			<div
				class="bg-slate-50 text-slate-900 max-h-48 overflow-y-scroll rounded-md"
			>
				<div
					class="text-sm font-mono select-all whitespace-pre mt-1 p-2"
				>
					{root}
				</div>
			</div>
		</div>

		<div class="overflow-x-hidden w-full">
			<div class="font-bold">Leaves</div>
			<div
				class="bg-slate-50 text-slate-900 max-h-48 overflow-y-scroll rounded-md"
			>
				<div
					class="text-sm font-mono select-all whitespace-pre mt-1 p-2"
				>
					{JSON.stringify(leaves, null, "  ")}
				</div>
			</div>
		</div>

		<div class="overflow-x-hidden w-full">
			<div class="font-bold">Legend</div>
			<div
				class="bg-slate-50 text-slate-900 max-h-48 overflow-y-scroll rounded-md"
			>
				<div
					class="text-sm font-mono select-all whitespace-pre mt-1 p-2"
				>
					{JSON.stringify(legend, null, "  ")}
				</div>
			</div>
		</div>
	</div>
</main>
