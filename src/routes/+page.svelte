<script>
	import Authenticator from '$lib/components/Authenticator.svelte';
	import SecretGenerator from '$lib/components/SecretGenerator.svelte';
	import Verifier from '$lib/components/Verifier.svelte';
</script>

<h1 class="text-2xl">TOTP tester</h1>

<div class="space-y-6">
	<div class="space-y-2">
		<h2 class="text-lg font-bold">Secret generator</h2>
		<SecretGenerator />
	</div>

	<div class="space-y-2">
		<h2 class="text-lg font-bold">Authenticator</h2>
		<Authenticator />
	</div>

	<div class="space-y-2">
		<h2 class="text-lg font-bold">Verifier</h2>
		<Verifier />
	</div>
</div>
