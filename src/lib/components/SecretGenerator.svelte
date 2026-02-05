<script lang="ts">
	import { TOTP } from '@otplib/totp';
	import { crypto } from '@otplib/plugin-crypto-web';
	import { base32 } from '@otplib/plugin-base32-scure';
	import QRCode from 'qrcode';
	import type { Attachment } from 'svelte/attachments';
	import CopyText from './CopyText.svelte';

	const localStoragePrefix = 'SecretGenerator_';

	let label = $state('');
	let issuer = $state('');
	let period = $state(30);
	let algorithm = $state('sha1');
	let digits = $state(6);
	let initialized = $state(false);
	let secret = $state('');
	let qrDataUrl = $state('');

	const localStorageAttachment: Attachment = () => {
		label = window.localStorage.getItem(`${localStoragePrefix}label`) ?? label;
		issuer = window.localStorage.getItem(`${localStoragePrefix}issuer`) ?? issuer;
		const savedPeriod = window.localStorage.getItem(`${localStoragePrefix}period`);
		if (savedPeriod) {
			period = Number(savedPeriod);
		}
		algorithm = window.localStorage.getItem(`${localStoragePrefix}algorithm`) ?? algorithm;
		const savedDigits = window.localStorage.getItem(`${localStoragePrefix}digits`);
		if (savedDigits) {
			digits = Number(savedDigits);
		}
		initialized = true;
	};

	$effect(() => {
		if (!initialized) {
			return;
		}

		window.localStorage.setItem(`${localStoragePrefix}label`, label);
		window.localStorage.setItem(`${localStoragePrefix}issuer`, issuer);
		window.localStorage.setItem(`${localStoragePrefix}period`, period.toString());
		window.localStorage.setItem(`${localStoragePrefix}algorithm`, algorithm);
		window.localStorage.setItem(`${localStoragePrefix}digits`, digits.toString());
	});

	async function generateSecret() {
		secret = '';
		qrDataUrl = '';
		const totp = new TOTP({
			crypto,
			base32,
			period,
			algorithm: algorithm as 'sha1' | 'sha256' | 'sha512',
			digits: digits as 6 | 7 | 8
		});
		const newSecret = totp.generateSecret();
		secret = newSecret;

		const uri = totp.toURI({
			secret: newSecret,
			label: label.trim() || 'Account',
			issuer: issuer.trim() || 'Issuer'
		});
		qrDataUrl = await QRCode.toDataURL(uri, {
			width: 200,
			margin: 1,
			color: { dark: '#000000', light: '#ffffff' }
		});
	}
</script>

<div {@attach localStorageAttachment}>
	<div class="flex max-w-96 flex-col gap-2">
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}label`}>Label</label>
			<input
				bind:value={label}
				placeholder="test@example.com"
				class="rounded-sm border border-stone-500 px-2 py-0.5"
				id={`${localStoragePrefix}label`}
			/>
		</div>
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}issuer`}>Issuer</label>
			<input
				bind:value={issuer}
				placeholder="My App"
				class="rounded-sm border border-stone-500 px-2 py-0.5"
				id={`${localStoragePrefix}issuer`}
			/>
		</div>
		<div class="flex flex-col gap-1">
			<label class="text-sm font-medium" for={`${localStoragePrefix}period`}>Period</label>
			<input
				bind:value={period}
				type="number"
				min={1}
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
				min={6}
				max={8}
				placeholder="6"
				class="rounded-sm border border-stone-500 px-2 py-0.5"
				id={`${localStoragePrefix}digits`}
			/>
		</div>
		<button
			class="rounded-sm border border-stone-500 bg-stone-200 px-2 py-0.5 transition-colors hover:bg-stone-100"
			onclick={generateSecret}
		>
			Generate secret
		</button>
		{#if secret}
			<div class="flex flex-col gap-2">
				<CopyText text={secret} />
				{#if qrDataUrl}
					<div class="flex flex-col gap-1">
						<span class="text-sm font-medium">QR code</span>
						<img
							src={qrDataUrl}
							alt="TOTP setup QR code"
							class="inline-block size-[200px] rounded border border-stone-500"
							width="200"
							height="200"
						/>
					</div>
				{/if}
			</div>
		{/if}
	</div>
</div>
