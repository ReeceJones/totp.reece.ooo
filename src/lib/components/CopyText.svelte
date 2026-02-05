<script lang="ts">
	let { text } = $props();

	let isCopied = $state(false);

	function copyText() {
		navigator.clipboard.writeText(text);
		isCopied = true;
		setTimeout(() => {
			isCopied = false;
		}, 1000);
	}
</script>

<div class="flex items-center justify-between gap-2 rounded-md border border-stone-500 px-2 py-1">
	<pre class="text-sm text-stone-500">{text}</pre>
	<button
		class="rounded-sm border border-stone-500 px-2 py-0.5 transition-colors hover:bg-stone-100"
		onclick={copyText}
	>
		{#if isCopied}
			<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
			</svg>
		{:else}
			<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"
				/>
			</svg>
		{/if}
	</button>
</div>
