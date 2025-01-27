<script lang="ts">
	import { onMount } from 'svelte'

	let disks = 3
	let moves = 0
	let towers = [Array.from({ length: disks }, (_, i) => disks - i), [], []]
	let draggingDisk: number | null = null
	let draggingTower: number | null = null
	let gameWon = false
	let timer = 0
	let timerInterval: number
	let isDragging = false

	// Ensure towers array is always valid
	$: {
		if (!Array.isArray(towers) || towers.length !== 3) {
			towers = [Array.from({ length: disks }, (_, i) => disks - i), [], []]
		}
	}

	onMount(() => {
		startTimer()
	})

	function startTimer() {
		timerInterval = setInterval(() => {
			timer++
		}, 1000)
	}

	function formatTime(seconds: number) {
		const mins = Math.floor(seconds / 60)
		const secs = seconds % 60
		return `${mins}:${secs.toString().padStart(2, '0')}`
	}

	function resetGame() {
		towers = [Array.from({ length: disks }, (_, i) => disks - i), [], []]
		moves = 0
		gameWon = false
		timer = 0
		clearInterval(timerInterval)
		startTimer()
	}

	function handleDragStart(
		event: DragEvent,
		towerIndex: number,
		diskIndex: number,
	) {
		if (!event.dataTransfer || towers[towerIndex].length === 0) return
		const tower = towers[towerIndex]
		if (diskIndex !== tower.length - 1) return

		draggingTower = towerIndex
		draggingDisk = tower[tower.length - 1]
		isDragging = true
		event.dataTransfer.effectAllowed = 'move'
		event.dataTransfer.setData('text/plain', `${towerIndex}`)

		if (event.target instanceof HTMLElement) {
			event.target.style.opacity = '0.4'
		}
	}

	function handleDragEnd(event: DragEvent) {
		isDragging = false
		if (event.target instanceof HTMLElement) {
			event.target.style.opacity = '1'
		}
	}

	function handleDragOver(event: DragEvent) {
		event.preventDefault()
		if (event.dataTransfer) {
			event.dataTransfer.dropEffect = 'move'
		}
	}

	function handleDrop(event: DragEvent, targetTowerIndex: number) {
		event.preventDefault()
		if (!event.dataTransfer) return

		const sourceTowerIndex = parseInt(event.dataTransfer.getData('text/plain'))
		if (canMoveDisk(sourceTowerIndex, targetTowerIndex)) {
			moveDisk(sourceTowerIndex, targetTowerIndex)
			moves++
			checkWin()
		}

		draggingTower = null
		draggingDisk = null
	}

	function canMoveDisk(from: number, to: number) {
		if (from < 0 || from >= towers.length || to < 0 || to >= towers.length)
			return false
		if (from === to) return false

		const fromTower = towers[from]
		const toTower = towers[to]

		if (!Array.isArray(fromTower) || !Array.isArray(toTower)) return false
		if (fromTower.length === 0) return false
		if (toTower.length === 0) return true

		return fromTower[fromTower.length - 1] < toTower[toTower.length - 1]
	}

	function moveDisk(from: number, to: number) {
		const disk = towers[from].pop()
		if (disk) towers[to].push(disk)
		towers = towers
	}

	function checkWin() {
		if (
			towers[2].length === disks &&
			towers[2].every((disk, i) => disk === disks - i)
		) {
			gameWon = true
			clearInterval(timerInterval)
		}
	}

	$: optimalMoves = Math.pow(2, disks) - 1
</script>

<main
	class="max-w-4xl mx-auto p-8 text-center bg-gradient-to-br from-yellow-300 to-orange-300 rounded-3xl shadow-xl"
>
	<h1 class="text-4xl font-bold text-gray-800 mb-8 text-shadow">
		Tower of Hanoi
	</h1>

	<div class="flex justify-between items-center my-8">
		<div class="text-left space-y-2">
			<p class="text-lg font-medium">Moves: {moves}</p>
			<p class="text-lg font-medium">Time: {formatTime(timer)}</p>
			<p class="text-lg font-medium">Optimal moves: {optimalMoves}</p>
		</div>

		<div class="flex gap-4 items-center">
			<label class="flex items-center gap-2">
				<span class="text-lg font-medium">Disks:</span>
				<input
					type="number"
					min="3"
					max="8"
					bind:value={disks}
					on:change={resetGame}
					class="w-20 p-3 border-2 border-blue-500 rounded-lg focus:outline-none focus:border-blue-600 shadow-md transition-all"
				/>
			</label>
			<button
				on:click={resetGame}
				class="px-6 py-3 bg-gradient-to-r from-blue-500 to-blue-600 text-white font-bold rounded-lg shadow-md hover:shadow-lg hover:-translate-y-0.5 transition-all"
				>Reset Game</button
			>
		</div>
	</div>

	<div class="flex justify-around items-end h-[300px] my-8">
		{#each towers as tower, towerIndex}
			<div
				class="relative w-[200px] h-[200px] cursor-pointer transition-colors duration-300 {isDragging &&
				canMoveDisk(draggingTower ?? -1, towerIndex)
					? 'bg-green-500/20'
					: ''}"
				on:dragover={handleDragOver}
				on:drop={(e) => handleDrop(e, towerIndex)}
			>
				<div
					class="absolute left-1/2 bottom-0 w-[10px] h-[200px] bg-gray-600 -translate-x-1/2"
				></div>
				<div
					class="absolute bottom-0 left-0 right-0 flex flex-col-reverse items-center"
				>
					{#each tower as disk, diskIndex}
						<div
							class="h-5 bg-blue-500 rounded-lg my-0.5 transition-all duration-300 ease-in-out cursor-grab hover:-translate-y-0.5 hover:shadow-lg hover:shadow-blue-500/30 {draggingTower ===
								towerIndex && diskIndex === tower.length - 1
								? 'opacity-40 scale-105 cursor-grabbing'
								: ''}"
							draggable={diskIndex === tower.length - 1}
							on:dragstart={(e) => handleDragStart(e, towerIndex, diskIndex)}
							on:dragend={handleDragEnd}
							style="width: {disk * 30 + 30}px;"
						></div>
					{/each}
				</div>
			</div>
		{/each}
	</div>

	{#if gameWon}
		<div
			class="fixed top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-white p-8 rounded-xl shadow-2xl"
		>
			<h2 class="text-2xl font-bold mb-4">Congratulations!</h2>
			<p class="text-lg">
				You solved the puzzle in {moves} moves and {formatTime(timer)}!
			</p>
		</div>
	{/if}
</main>
