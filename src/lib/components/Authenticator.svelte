<script lang="ts">
	import { generate } from '@otplib/totp';
	import { crypto } from '@otplib/plugin-crypto-web';
	import { base32 } from '@otplib/plugin-base32-scure';
	import type { Attachment } from 'svelte/attachments';
	import CopyText from './CopyText.svelte';

	const localStoragePrefix = 'Verifier_';

	let secret = $state('');
	let period = $state(30);
	let algorithm = $state('sha1');
	let digits = $state(6);
	let initialized = $state(false);
	let totpCode = $state('');

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

	async function generateTOTPCode() {
		totpCode = await generate({
			secret,
			crypto,
			base32,
			period,
			algorithm: algorithm as 'sha1' | 'sha256' | 'sha512',
			digits: digits as 6 | 7 | 8
		});
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
		<button
			class="rounded-sm border border-stone-500 bg-stone-200 px-2 py-0.5 transition-colors hover:bg-stone-100"
			onclick={generateTOTPCode}
		>
			Generate TOTP code
		</button>
		{#if totpCode}
			<CopyText text={totpCode} />
		{/if}
	</div>
</div>
