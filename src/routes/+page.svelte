<script lang="ts">
	import { fade } from 'svelte/transition';

	let inputUrl = '';
	let minimal = '';
	let passphrase = '';
	let streamId = '';
	let copied = '';
	let showCopied = false;

	function doParseUrl() {
		try {
			const url = new URL(inputUrl);
			const searchParams = new URLSearchParams(url.search);

			passphrase = searchParams.get('passphrase') ?? '';
			streamId = searchParams.get('streamid') ?? '';

			const minimalParams = new URLSearchParams();
			const transtype = searchParams.get('transtype');
			if (transtype) minimalParams.set('transtype', transtype);

			minimal = `${url.protocol}//${url.host}?${minimalParams.toString()}`;
		} catch {
			minimal = '';
			passphrase = '';
			streamId = '';
		}
	}

	async function copy(text: string, label: string) {
		try {
			await navigator.clipboard.writeText(text);
			copied = `✅ Copied ${label}`;
			showCopied = true;
			setTimeout(() => {
				showCopied = false;
			}, 2000);
		} catch {
			copied = '❌ Copy failed';
			showCopied = true;
			setTimeout(() => {
				showCopied = false;
			}, 2000);
		}
	}

	async function pasteFromClipboard() {
		try {
			const text = await navigator.clipboard.readText();
			inputUrl = text;
		} catch {
			copied = '❌ Could not read clipboard';
			showCopied = true;
			setTimeout(() => {
				showCopied = false;
			}, 2000);
		}
	}

	$: inputUrl, doParseUrl();
</script>

<main class="mx-auto max-w-xl p-4 font-sans text-gray-900 sm:p-8">
	<div class="mb-8 flex flex-col items-center">
		<div class="flex items-center gap-2">
			<span class="text-3xl sm:text-4xl" aria-hidden="true">🎥</span>
			<h1
				class="text-2xl font-extrabold tracking-tight sm:text-3xl"
				id="srt-parser-heading"
				tabindex="-1"
			>
				SRT Parser
			</h1>
		</div>
		<p
			class="mt-2 max-w-md text-center text-base text-gray-500 sm:text-lg"
			id="srt-parser-instructions"
		>
			Paste your full SRT link. We’ll extract the key parts you need to input into <a
				class="text-blue-600 underline"
				href="https://play.google.com/store/apps/details?id=app.irlpro.android">IRL Pro</a
			>’s settings to get it working.
		</p>
	</div>

	<div class="mb-8">
		<label for="url" class="mb-2 block text-sm font-semibold text-gray-700">
			Paste full SRT link:
		</label>
		<div class="flex gap-2">
			<input
				id="url"
				type="text"
				bind:value={inputUrl}
				placeholder="srt://some.domain.com:6000?...&passphrase=...&streamid=..."
				class="flex-1 rounded-lg border border-gray-300 px-4 py-2 text-sm shadow-sm transition focus:border-blue-500 focus:ring-2 focus:ring-blue-200 focus:outline-none"
				aria-describedby="srt-parser-instructions"
				aria-label="Paste full SRT link"
				autocomplete="off"
			/>
			<button
				on:click={pasteFromClipboard}
				type="button"
				class="inline-flex items-center gap-2 rounded-lg border border-gray-300 bg-gray-50 px-3 py-2 text-sm font-medium text-gray-700 transition hover:bg-gray-100 active:bg-gray-200"
				aria-label="Paste SRT link from clipboard"
			>
				<span class="text-lg" aria-hidden="true">📋</span> Paste
			</button>
		</div>
	</div>

	{#if minimal || passphrase || streamId}
		<div class="animate-fade-in space-y-6" aria-live="polite" aria-atomic="true">
			{#if minimal}
				<div
					in:fade={{ duration: 300 }}
					out:fade={{ duration: 300 }}
					class="rounded-xl border border-blue-100 bg-blue-50/60 p-4 shadow-sm"
					role="region"
					aria-labelledby="streaming-url-heading"
				>
					<div class="mb-1 flex items-center justify-between">
						<h2
							id="streaming-url-heading"
							class="flex items-center gap-1 font-semibold text-blue-700"
						>
							<span class="text-lg" aria-hidden="true">📡</span> Streaming Server (URL)
						</h2>
						<button
							type="button"
							on:click={() => copy(minimal, 'URL')}
							class="inline-flex items-center gap-1 rounded-lg bg-blue-100 px-3 py-1.5 text-xs font-medium text-blue-700 transition hover:bg-blue-200 active:bg-blue-300"
							aria-label="Copy streaming server URL to clipboard"
						>
							<span class="text-base" aria-hidden="true">📄</span> Copy
						</button>
					</div>
					<pre
						class="mt-2 overflow-x-auto rounded-md bg-gray-900 p-3 font-mono text-sm break-all text-gray-400"
						tabIndex="0"><code>{minimal}</code></pre>
				</div>
			{/if}

			{#if passphrase}
				<div
					in:fade={{ duration: 300 }}
					out:fade={{ duration: 300 }}
					class="rounded-xl border border-amber-100 bg-amber-50/60 p-4 shadow-sm"
					role="region"
					aria-labelledby="stream-passphrase-heading"
				>
					<div class="mb-1 flex items-center justify-between">
						<h2
							id="stream-passphrase-heading"
							class="flex items-center gap-1 font-semibold text-amber-700"
						>
							<span class="text-lg" aria-hidden="true">🔐</span> Stream Passphrase
						</h2>
						<button
							type="button"
							on:click={() => copy(passphrase, 'Passphrase')}
							class="inline-flex items-center gap-1 rounded-lg bg-amber-100 px-3 py-1.5 text-xs font-medium text-amber-700 transition hover:bg-amber-200 active:bg-amber-300"
							aria-label="Copy stream passphrase to clipboard"
						>
							<span class="text-base" aria-hidden="true">📄</span> Copy
						</button>
					</div>
					<pre
						class="mt-2 overflow-x-auto rounded-md bg-gray-900 p-3 font-mono text-sm break-all text-gray-400"
						tabIndex="0"><code>{passphrase}</code></pre>
				</div>
			{/if}

			{#if streamId}
				<div
					in:fade={{ duration: 300 }}
					out:fade={{ duration: 300 }}
					class="rounded-xl border border-purple-100 bg-purple-50/60 p-4 shadow-sm"
					role="region"
					aria-labelledby="stream-id-heading"
				>
					<div class="mb-1 flex items-center justify-between">
						<h2
							id="stream-id-heading"
							class="flex items-center gap-1 font-semibold text-purple-700"
						>
							<span class="text-lg" aria-hidden="true">🧬</span> Stream ID
						</h2>
						<button
							type="button"
							on:click={() => copy(streamId, 'Stream ID')}
							class="inline-flex items-center gap-1 rounded-lg bg-purple-100 px-3 py-1.5 text-xs font-medium text-purple-700 transition hover:bg-purple-200 active:bg-purple-300"
							aria-label="Copy stream ID to clipboard"
						>
							<span class="text-base" aria-hidden="true">📄</span> Copy
						</button>
					</div>
					<pre
						class="mt-2 overflow-x-auto rounded-md bg-gray-900 p-3 font-mono text-sm break-all text-gray-400"
						tabIndex="0"><code>{streamId}</code></pre>
				</div>
			{/if}
		</div>
	{/if}

	{#if showCopied}
		<div
			class="fixed bottom-4 left-1/2 z-50 w-[90vw] -translate-x-1/2 transform rounded-lg bg-blue-600 px-4 py-2 text-white shadow-lg sm:w-[320px]"
			role="status"
			aria-live="polite"
			in:fade={{ duration: 300 }}
			out:fade={{ duration: 500 }}
		>
			<p class="text-center font-medium">{copied}</p>
		</div>
	{/if}
</main>

<style>
	@keyframes fade-in {
		from {
			opacity: 0;
			transform: translateY(16px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	.animate-fade-in {
		animation: fade-in 0.3s cubic-bezier(0.4, 0, 0.2, 1);
	}
</style>
