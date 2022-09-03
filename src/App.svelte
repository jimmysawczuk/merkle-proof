<script>
	import { csvParse } from "d3"
	import { ethers } from "ethers"
	import { MerkleTree } from "merkletreejs"
	import "buffer"

	const availableTypes = ["address", "uint256", "uint64"]

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

		const ty = headers.map((header) => {
			return types[header]
		})

		for (const row of data) {
			const r = headers.map((header) => row[header])
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

<main class="w-screen h-screen">
	<div class="w-full h-full grid grid-cols-[2fr,1fr]">
		<div class="grid grid-rows-[2fr,3fr] gap-4 p-4">
			<div class="flex flex-col">
				<p class="flex-none text-lg">Paste your CSV here:</p>
				<div class="flex-grow">
					<textarea
						class="w-full h-full resize-none block rounded-md text-sm font-mono p-2 text-slate-900 border border-slate-900/60 dark:border-slate-50/60 transition-colors"
						bind:value={rawCSV}
						on:keyup={parseCSV}
					/>
				</div>
			</div>

			<div class="overflow-y-scroll">
				{#if data.length > 0}
					<table
						class="w-full overflow-hidden shadow-lg dark:shadow-none border border-slate-900"
					>
						<thead class="text-left">
							<tr class="bg-sky-800">
								{#each headers as header}
									<th class="text-lg p-1">
										<div class="text-white">{header}</div>
										<select
											bind:value={types[header]}
											on:change={updateRoot}
											class="font-normal block w-full py-1 bg-slate-50 dark:bg-slate-900 transition-colors"
										>
											<option value={null} disabled
												>Select a type</option
											>
											{#each availableTypes as ty}
												<option value={ty}>{ty}</option>
											{/each}
										</select>
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

		<div class="space-y-6 p-4 bg-sky-900 text-white">
			<div class="bg-slate-900/50 p-4 rounded-md">
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

			<div>
				<div class="font-bold">Root</div>
				<div
					class="text-sm font-mono select-all whitespace-pre mt-1 py-4 px-2 overflow-x-hidden bg-slate-50 text-slate-900 rounded-md"
				>
					{root}
				</div>
			</div>

			<div>
				<div class="font-bold">Leaves</div>
				<div
					class="text-sm font-mono select-all whitespace-pre mt-1 py-4 px-2 overflow-x-hidden bg-slate-50 text-slate-900 rounded-md"
				>
					{JSON.stringify(leaves, null, "  ")}
				</div>
			</div>

			<div>
				<div class="font-bold">Legend</div>
				<div
					class="text-sm font-mono select-all whitespace-pre mt-1 py-4 px-2 overflow-x-hidden bg-slate-50 text-slate-900 rounded-md"
				>
					{JSON.stringify(legend, null, "  ")}
				</div>
			</div>
		</div>
	</div>
</main>
