<script lang="ts">
	import { verify } from '@otplib/totp';
	import { crypto } from '@otplib/plugin-crypto-web';
	import { base32 } from '@otplib/plugin-base32-scure';
	import type { Attachment } from 'svelte/attachments';

	const localStoragePrefix = 'Verifier_';

	let secret = $state('');
	let period = $state(30);
	let algorithm = $state('sha1');
	let digits = $state(6);
	let token = $state('');
	let initialized = $state(false);
	let verifying = $state(false);
	let result = $state<{ valid: boolean; delta?: number } | null>(null);

	const localStorageAttachment: Attachment = () => {
		secret = window.localStorage.getItem(`${localStoragePrefix}secret`) ?? secret;
		period = Number(window.localStorage.getItem(`${localStoragePrefix}period`) ?? period);
		algorithm = window.localStorage.getItem(`${localStoragePrefix}algorithm`) ?? algorithm;
		digits = Number(window.localStorage.getItem(`${localStoragePrefix}digits`) ?? digits);
		initialized = true;
	};

	$effect(() => {
		if (!initialized) {
			return;
		}

		window.localStorage.setItem(`${localStoragePrefix}secret`, secret);
		window.localStorage.setItem(`${localStoragePrefix}period`, period.toString());
		window.localStorage.setItem(`${localStoragePrefix}algorithm`, algorithm);
		window.localStorage.setItem(`${localStoragePrefix}digits`, digits.toString());
	});

	async function verifyToken() {
		result = null;
		if (!token.trim()) {
			return;
		}
		verifying = true;
		try {
			const verifyResult = await verify({
				secret,
				token: token.trim(),
				crypto,
				base32,
				period,
				algorithm: algorithm as 'sha1' | 'sha256' | 'sha512',
				digits: digits as 6 | 7 | 8,
				epochTolerance: 1
			});
			result = {
				valid: verifyResult.valid,
				delta: 'delta' in verifyResult ? verifyResult.delta : undefined
			};
		} catch (err) {
			result = { valid: false };
		} finally {
			verifying = false;
		}
	}
</script>

<div {@attach localStorageAttachment}>
	<div class="flex max-w-96 flex-col gap-2">
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}secret`}>Secret</label>
			<input
				bind:value={secret}
				placeholder="1234567890"
				class="rounded-sm border border-stone-500 px-2 py-0.5"
				id={`${localStoragePrefix}secret`}
			/>
		</div>
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}period`}>Period</label>
			<input
				bind:value={period}
				type="number"
				min={0}
				placeholder="30"
				class="rounded-sm border border-stone-500 px-2 py-0.5"
				id={`${localStoragePrefix}period`}
			/>
		</div>
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}algorithm`}>Algorithm</label>
			<select
				bind:value={algorithm}
				class="rounded-sm border border-stone-500 px-2 py-0.5"
				id={`${localStoragePrefix}algorithm`}
			>
				<option value="sha1">SHA1</option>
				<option value="sha256">SHA256</option>
				<option value="sha512">SHA512</option>
			</select>
		</div>
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}digits`}>Digits</label>
			<input
				bind:value={digits}
				type="number"
				min={0}
				placeholder="6"
				class="rounded-sm border border-stone-500 px-2 py-0.5"
				id={`${localStoragePrefix}digits`}
			/>
		</div>
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}token`}>Code to verify</label>
			<input
				bind:value={token}
				type="text"
				inputmode="numeric"
				autocomplete="one-time-code"
				placeholder="000000"
				class="rounded-sm border border-stone-500 px-2 py-0.5 font-mono"
				id={`${localStoragePrefix}token`}
				onkeydown={(e) => e.key === 'Enter' && verifyToken()}
			/>
		</div>
		<button
			class="rounded-sm border border-stone-500 bg-stone-200 px-2 py-0.5 transition-colors hover:bg-stone-100 disabled:opacity-50"
			onclick={verifyToken}
			disabled={verifying || !token.trim()}
		>
			{verifying ? 'Verifying…' : 'Verify code'}
		</button>
		{#if result !== null}
			<div
				class="rounded-md border px-2 py-1 text-sm font-medium {result.valid
					? 'border-green-600 bg-green-50 text-green-800'
					: 'border-red-600 bg-red-50 text-red-800'}"
				role="status"
			>
				{#if result.valid}
					✓ Valid
					{#if result.delta !== undefined && result.delta !== 0}
						<span class="font-normal"> (delta: {result.delta})</span>
					{/if}
				{:else}
					✗ Invalid
				{/if}
			</div>
		{/if}
	</div>
</div>
