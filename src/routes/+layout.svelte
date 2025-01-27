<script lang="ts">
	import type { Snippet } from 'svelte'
	import '../app.css'
	import type { LayoutData } from './$types'

	let { data, children }: { data: LayoutData; children: Snippet } = $props()
</script>

<main class="min-h-screen bg-gray-100 p-4">
	{@render children()}
</main>

<style>
	:global(body) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}
</style>
