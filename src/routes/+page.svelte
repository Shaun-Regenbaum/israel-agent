<script lang="ts">
	import Realtime from '$lib/realtime/Realtime.svelte';
	import Switch from '$lib/ui/Switch.svelte';
	import Layout from '$lib/ui/Layout.svelte';
	import type { ItemType } from '@openai/realtime-api-beta/dist/lib/client';
	import { onMount } from 'svelte';
	import WebcamStream from '$lib/ui/WebcamStream.svelte';
	import { PUBLIC_OPENAI_API_KEY } from '$env/static/public';


	let startConversation: () => Promise<void>;
	let endConversation: () => Promise<void>;

	let turnDetection: 'server_vad' | 'none' = 'server_vad';

	let items: ItemType[] = [];

	let isConnected = false;
	let isRecording = false;

	let startRecording: () => Promise<void>;
	let stopRecording: () => Promise<void>;

	let apiKey = '';

	let contextInstructions = '';

	onMount(() => {
		apiKey = PUBLIC_OPENAI_API_KEY;
	});
</script>

<Layout>
	<div slot="header" class="p-6 text-white text-sm font-semibold border-b border-white/15 w-full">
		<div class="max-w-7xl mx-auto flex justify-between">
			<div>svelte openai realtime api</div>
		</div>
	</div>

	<div class="flex gap-6 p-6 mx-auto max-w-7xl h-full">
		<!-- Left Column - Video and Context -->
		<div class="w-[600px] flex flex-col gap-6">
			<div class="webcam-wrapper">
				<WebcamStream />
			</div>
			
			<div class="context-form bg-white/5 rounded-lg p-4">
				<label for="context" class="block text-sm font-medium text-white mb-2">
					Context Instructions
				</label>
				<textarea
					id="context"
					bind:value={contextInstructions}
					rows="4"
					class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
					placeholder="Add any specific instructions or context for the AI..."
				/>
			</div>
		</div>

		<!-- Right Column - Transcript -->
		<div class="flex-1 bg-white/5 rounded-lg p-4 overflow-y-auto max-h-[calc(100vh-12rem)]">
			{#key items}
				{#each items as item, index}
					{#if item.formatted && index > 0}
						<div
							class="text-sm py-4 border-b border-white/5 {item.role === 'user'
								? 'text-amber-200'
								: 'text-white'}"
						>
							<span class="font-semibold">{item.role}:</span>
							{item.role === 'user'
								? item.formatted.transcript || '(empty)'
								: item.formatted.transcript || '(truncated)'}
						</div>
					{/if}
				{/each}
			{/key}
		</div>
	</div>

	<div slot="footer" class="p-6 py-4 border-t border-white/15 w-full">
		<div class="items-center flex justify-between w-full max-w-7xl mx-auto">
			<div class="text-white text-sm font-semibold flex flex-col gap-3">
				<div>turn detection</div>
				<Switch options={['server_vad', 'none']} bind:selected={turnDetection} />
			</div>

			{#if turnDetection === 'none' && isConnected}
				<button
					type="button"
					on:pointerdown={() => {
						startRecording();
					}}
					on:pointerup={() => {
						stopRecording();
					}}
					class="rounded-full px-5 py-3 text-base font-semibold focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-amber-500 text-black bg-amber-500 hover:bg-amber-400 active:bg-amber-700"
					>press and hold</button
				>
			{/if}

			<button
				type="button"
				on:click={() => {
					if (!apiKey) {
						alert('Please set your OpenAI API Key first');
						return;
					}

					if (isConnected) {
						endConversation();
					} else {
						startConversation();
					}
				}}
				class="rounded-full px-5 py-3 text-base font-semibold focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-amber-500 {isConnected
					? 'text-amber-500 bg-amber-500/10 border border-amber-500/20 hover:bg-amber-500/20'
					: 'text-black bg-amber-500 hover:bg-amber-400'} {apiKey
					? ''
					: 'cursor-not-allowed opacity-50'}">{isConnected ? 'disconnect' : 'connect'}</button
			>
		</div>
	</div>
</Layout>

{#if apiKey}
	<Realtime
		bind:startConversation
		bind:endConversation
		bind:isConnected
		bind:isRecording
		bind:items
		bind:startRecording
		bind:stopRecording
		{turnDetection}
		{apiKey}
	/>
{/if}

<style>
	.webcam-wrapper {
		width: 100%;
		background: rgba(0, 0, 0, 0.2);
		border-radius: 0.5rem;
		overflow: hidden;
	}

	textarea {
		resize: vertical;
		min-height: 100px;
	}

	:global(body) {
		background: #000;
	}
</style>
