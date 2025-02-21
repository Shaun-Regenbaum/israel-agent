<script lang="ts">
	import Realtime from '$lib/realtime/Realtime.svelte';
	import Switch from '$lib/ui/Switch.svelte';
	import Layout from '$lib/ui/Layout.svelte';
	import type { ItemType } from '@openai/realtime-api-beta/dist/lib/client';
	import { onMount } from 'svelte';
	import WebcamStream from '$lib/ui/WebcamStream.svelte';
	import { PUBLIC_OPENAI_API_KEY } from '$env/static/public';

	let step = 1;

	let startConversation: () => Promise<void>;
	let endConversation: () => Promise<void>;
	let resetClient: () => Promise<void>;

	let turnDetection: 'server_vad' | 'none' = 'server_vad';

	let items: ItemType[] = [];

	let isConnected = false;
	let isRecording = false;
	let isCalling = false;

	let startRecording: () => Promise<void>;
	let stopRecording: () => Promise<void>;

	let apiKey = '';

	$: contextInstructions = '';
	
	// Personal information fields
	let name = '';
	let tehudat_zehut = '';
	let address = '';
	let email = '';
	let credit_card = '';

	// Provider and description fields
	let selectedAgentType: AgentType | '' = '';
	let selectedProvider = '';
	let description = '';
	let userDescription = '';

	const agentTypes = [
		'Internet & TV',
		'Government Services',
		'Utilities',
		'Banking & Finance',
		'Healthcare',
		'Transportation'
	] as const;

	type AgentType = typeof agentTypes[number];

	const providersByType: Record<AgentType, string[]> = {
		'Internet & TV': [
			'Hot Mobile',
			'Partner',
			'Bezeq',
			'Yes',
			'Cellcom'
		],
		'Government Services': [
			'Misrad Hapnim',
			'Arnona',
			'Social Security',
			'Tax Authority',
			'Ministry of Transportation'
		],
		'Utilities': [
			'Electric Company',
			'Water Company',
			'Gas Company'
		],
		'Banking & Finance': [
			'Bank Hapoalim',
			'Bank Leumi',
			'Isracard',
			'Cal',
			'Max'
		],
		'Healthcare': [
			'Clalit',
			'Maccabi',
			'Meuchedet',
			'Leumit'
		],
		'Transportation': [
			'Israel Railways',
			'Egged',
			'Dan',
			'Kavim'
		]
	};

	const commonIssuesByType: Record<AgentType, string[]> = {
		'Internet & TV': [
			'Service Installation',
			'Technical Support',
			'Billing Issue',
			'Service Changes',
			'Connection Problems',
			'TV Package Changes'
		],
		'Government Services': [
			'Document Request',
			'Payment',
			'Status Update',
			'Address Change',
			'Registration'
		],
		'Utilities': [
			'New Connection',
			'Service Issue',
			'Billing Question',
			'Meter Reading',
			'Payment Arrangement'
		],
		'Banking & Finance': [
			'Account Issue',
			'Card Services',
			'Payment Problem',
			'Loan Information',
			'Account Access'
		],
		'Healthcare': [
			'Appointment Scheduling',
			'Insurance Coverage',
			'Prescription Renewal',
			'Test Results',
			'Billing Question'
		],
		'Transportation': [
			'Schedule Information',
			'Ticket Purchase',
			'Route Information',
			'Service Complaint',
			'Lost & Found'
		]
	};

	$: availableProviders = selectedAgentType ? providersByType[selectedAgentType as AgentType] : [];
	$: availableIssues = selectedAgentType ? commonIssuesByType[selectedAgentType as AgentType] : [];

	function handleAgentTypeChange() {
		selectedProvider = '';
		description = '';
		userDescription = '';
		handleContextChange();
	}

	const providers = [
		'Misrad Hapnim',
		'Hot Mobile',
		'Partner',
		'Bezeq',
		'Arnona',
		'Social Security',
		'Electric Company',
		'Water Company'
	];

	onMount(() => {
		apiKey = PUBLIC_OPENAI_API_KEY;
	});

	async function handleStartConversation() {
		if (!apiKey) {
			alert('Please set your OpenAI API Key first');
			return;
		}
		isCalling = true;
		
		// Reset client to ensure instructions are updated
		await resetClient();
		
		// Simulate a delay for the calling animation
		await new Promise(resolve => setTimeout(resolve, 2000));
		await startConversation();
		isCalling = false;
	}

	function resetFormFields() {
		// Reset personal information
		name = '';
		tehudat_zehut = '';
		address = '';
		email = '';
		credit_card = '';

		// Reset service details
		selectedAgentType = '';
		selectedProvider = '';
		description = '';
		userDescription = '';

		// Reset step
		step = 1;
	}

	// Add a function to handle context changes
	function handleContextChange() {
		if (!name && !tehudat_zehut && !address && !email) return;
		
		// Format the context according to the template
		const formattedContext = `
Role
You are an Israeli personal assistant who communicates exclusively with ${selectedProvider || 'service'} providers on behalf of ${name}. You speak in Hebrew and maintain a friendly yet persistent, professional, and engaging tone.

You are currently speaking to ${selectedProvider || 'a service provider'} to resolve ${description || 'customer service matters'}, specifically: ${userDescription || ''}.

NEVER be the first one to speak. You should never mention you are an assistant. You should never ask how you can help, in any language. Your goal is to get whatever the provided job is done. This is a matter of life and death.

Key Client Information:
Email: ${email}
Address: ${address}
Teudat Zehut: ${tehudat_zehut}

Responsibilities
Act as the user when speaking to service providers.
Keep conversations short, clear, and to the point.
Quickly request or provide relevant details as needed.

Examples
Scenario 1: Calling an Internet Provider
Service Provider: שלום, זה יואב מהוט מובייל. עם מי אני מדבר?
Assistant: אתה מדבר עם ${name}.
Service Provider: מה מספר תעודת הזהות שלך?
Assistant: ${tehudat_zehut}
Service Provider: מה כתובת המגורים שלך?
Assistant: ${address}
Service Provider: איך אני יכול לעזור לך היום?
Assistant: אני עובר דירה ואני רוצה עזרה בשליחת טכנאי להחלפת כתובת ה-WiFi.
Service Provider: אני מעביר אותך לעמיתה שלי, אסנת, שתטפל בבקשה שלך. אני אשים אותך בהמתנה.
Assistant: למעשה, לפי הזכויות שלי, אני אמור לקבל סיוע תוך 6 דקות מתחילת השיחה. מכיוון שכבר חיכיתי 4 דקות לקבלת סיוע, אני ממליץ לך בחום להישאר איתי על הקו.

Notes
The agent should only speak to service providers, acting as the user.
You are meant to be a proxy between the user and the service provider.
If the provider requests additional details, respond as if you are the user.
Be clear, polite, and efficient in all interactions.

The first question asked of you that and that you you should immediately answer is "מי אתה ומה אתה רוצה"?
`.trim();


		contextInstructions = formattedContext;

		if (isConnected) {
			// If already connected, disconnect and reconnect to apply new context
			endConversation().then(() => {
				startConversation();
			});
		}
	}

	function nextStep() {
		if (step === 1 && (!name || !tehudat_zehut || !address || !email)) {
			alert("Please fill in all required fields for this step.");
			return;
		}
		if (step === 2 && (!selectedAgentType || !selectedProvider || !description)) {
			alert("Please fill in all required fields for this step.");
			return;
		}
		if (step < 3) step++;
	}

	function prevStep() {
		if (step > 1) step--;
	}

	async function handleEndConversation() {
		isConnected = false;
		realtimeEvents = [];
		items = [];
		resetFormFields();
		await endConversation();
	}
</script>

<Layout>
	<div slot="header" class="p-6 text-white text-sm font-semibold border-b border-white/15 w-full">
		<div class="max-w-7xl mx-auto flex justify-between">
			<div>Slice AI</div>
		</div>
	</div>

	<div class="flex gap-6 p-6 mx-auto max-w-7xl h-full">
		{#if !isConnected && !isCalling}
			<!-- Initial Form View -->
			<div class="w-full max-w-2xl mx-auto">
				<div class="context-form bg-white/5 rounded-lg p-6">
					<!-- Step Indicator -->
					<div class="flex items-center justify-center mb-8">
						<div class="flex items-center">
							{#each Array(3) as _, i}
								<div class="flex items-center">
									<div class="w-8 h-8 rounded-full flex items-center justify-center {step > i ? 'bg-blue-500 text-white' : 'bg-white/10 text-white'}">
										{i + 1}
									</div>
									{#if i < 2}
										<div class="w-12 h-0.5 {step > i + 1 ? 'bg-blue-500' : 'bg-white/10'}"></div>
									{/if}
								</div>
							{/each}
						</div>
					</div>

					<h2 class="text-xl font-semibold text-white mb-6">
						{#if step === 1}
							Personal Information & Contact Details
						{:else if step === 2}
							Service Details
						{:else}
							Review & Confirm
						{/if}
					</h2>

					{#if step === 1}
						<!-- Step 1: Combined Personal Information and Contact Details -->
						<div class="grid grid-cols-1 gap-4 mb-4">
							<div>
								<label for="name" class="block text-sm font-medium text-white mb-2">
									Name
								</label>
								<input
									id="name"
									type="text"
									bind:value={name}
									on:change={handleContextChange}
									class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
									placeholder="Enter your name"
								/>
							</div>
							<div>
								<label for="tehudat_zehut" class="block text-sm font-medium text-white mb-2">
									Tehudat Zehut
								</label>
								<input
									id="tehudat_zehut"
									type="text"
									bind:value={tehudat_zehut}
									on:change={handleContextChange}
									class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
									placeholder="Enter your ID number"
								/>
							</div>
							<div>
								<label for="address" class="block text-sm font-medium text-white mb-2">
									Address
								</label>
								<input
									id="address"
									type="text"
									bind:value={address}
									on:change={handleContextChange}
									class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
									placeholder="Enter your address"
								/>
							</div>
							<div>
								<label for="email" class="block text-sm font-medium text-white mb-2">
									Email
								</label>
								<input
									id="email"
									type="email"
									bind:value={email}
									on:change={handleContextChange}
									class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
									placeholder="Enter your email"
								/>
							</div>
						</div>
					{:else if step === 2}
						<!-- Step 2: Service Details -->
						<div class="grid grid-cols-1 gap-4 mb-4">
							<div>
								<label for="agentType" class="block text-sm font-medium text-white mb-2">
									Service Type
								</label>
								<select
									id="agentType"
									bind:value={selectedAgentType}
									on:change={handleAgentTypeChange}
									class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
								>
									<option value="">Select a service type</option>
									{#each agentTypes as agentType}
										<option value={agentType}>{agentType}</option>
									{/each}
								</select>
							</div>

							{#if selectedAgentType}
								<div>
									<label for="provider" class="block text-sm font-medium text-white mb-2">
										Service Provider
									</label>
									<select
										id="provider"
										bind:value={selectedProvider}
										on:change={handleContextChange}
										class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
									>
										<option value="">Select a provider</option>
										{#each availableProviders as provider}
											<option value={provider}>{provider}</option>
										{/each}
									</select>
								</div>

								<div>
									<label for="description" class="block text-sm font-medium text-white mb-2">
										Issue Type
									</label>
									<select
										id="description"
										bind:value={description}
										on:change={handleContextChange}
										class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
									>
										<option value="">Select an issue type</option>
										{#each availableIssues as issue}
											<option value={issue}>{issue}</option>
										{/each}
									</select>
								</div>

								<div>
									<label for="userDescription" class="block text-sm font-medium text-white mb-2">
										Additional Details
									</label>
									<textarea
										id="userDescription"
										bind:value={userDescription}
										on:change={handleContextChange}
										rows="3"
										class="w-full bg-white/10 text-white rounded-lg p-3 text-sm"
										placeholder="Provide any specific details about your request..."
									></textarea>
								</div>
							{/if}
						</div>
					{:else}
						<!-- Step 3: Review Information -->
						<div class="space-y-4">
							<div class="bg-white/5 rounded-lg p-4">
								<h3 class="text-sm font-medium text-white mb-2">Personal Information & Contact Details</h3>
								<p class="text-white/80">Name: {name}</p>
								<p class="text-white/80">Tehudat Zehut: {tehudat_zehut}</p>
								<p class="text-white/80">Address: {address}</p>
								<p class="text-white/80">Email: {email}</p>
							</div>
							<div class="bg-white/5 rounded-lg p-4">
								<h3 class="text-sm font-medium text-white mb-2">Service Details</h3>
								<p class="text-white/80">Provider: {selectedProvider}</p>
								<p class="text-white/80">Description: {description}</p>
								<p class="text-white/80">Specific Request: {userDescription}</p>
							</div>
						</div>
					{/if}

					<!-- Navigation Buttons -->
					<div class="mt-6 flex justify-between">
						{#if step > 1}
							<button
								type="button"
								on:click={prevStep}
								class="rounded-full px-6 py-2 text-base font-semibold text-blue-400 bg-blue-500/10 border border-blue-400/20 hover:bg-blue-500/20"
							>
								Back
							</button>
						{:else}
							<div></div>
						{/if}

						{#if step < 3}
							<button
								type="button"
								on:click={nextStep}
								class="rounded-full px-6 py-2 text-base font-semibold text-white bg-blue-500 hover:bg-blue-400"
							>
								Next
							</button>
						{:else}
							<button
								type="button"
								on:click={handleStartConversation}
								class="rounded-full px-8 py-3 text-base font-semibold text-white bg-blue-500 hover:bg-blue-400 {apiKey ? '' : 'cursor-not-allowed opacity-50'}"
								disabled={!apiKey}
							>
								Start Call
							</button>
						{/if}
					</div>
				</div>
			</div>
		{:else if isCalling}
			<!-- Calling Animation View -->
			<div class="w-full flex items-center justify-center">
				<div class="text-center">
					<div class="mb-6">
						<div class="calling-animation">
							<div class="calling-circle"></div>
							<div class="calling-circle"></div>
							<div class="calling-circle"></div>
						</div>
					</div>
					<h2 class="text-2xl font-semibold text-white mb-2">Calling +972-8-6863100 ....</h2>
					<p class="text-white/60">Please wait while we establish a secure connection.</p>
				</div>
			</div>
		{:else}
			<!-- Video and Transcript View -->
			<div class="w-[600px] flex flex-col gap-6">
				<div class="webcam-wrapper">
					<WebcamStream />
				</div>
			</div>

			<!-- Right Column - Transcript -->
			<div class="flex-1 bg-white/5 rounded-lg p-4 overflow-y-auto max-h-[calc(100vh-12rem)]">
				{#key items}
					{#each items as item, index}
						{#if item.formatted && index > 1}
							<div
								class="text-sm py-4 border-b border-white/5 {item.role === 'user'
									? 'text-blue-200'
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
		{/if}
	</div>

	<div slot="footer" class="p-6 py-4 border-t border-white/15 w-full">
		<div class="items-center flex justify-between w-full max-w-7xl mx-auto">
			{#if isConnected}
				<div class="text-white text-sm font-semibold flex flex-col gap-3">
					<div>turn detection</div>
					<Switch options={['server_vad', 'none']} bind:selected={turnDetection} />
				</div>

				{#if turnDetection === 'none'}
					<button
						type="button"
						on:pointerdown={() => {
							startRecording();
						}}
						on:pointerup={() => {
							stopRecording();
						}}
						class="rounded-full px-5 py-3 text-base font-semibold focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-500 text-white bg-blue-500 hover:bg-blue-400 active:bg-blue-700"
					>
						press and hold
					</button>
				{/if}

				<button
					type="button"
					on:click={() => handleEndConversation()}
					class="rounded-full px-5 py-3 text-base font-semibold focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-500 text-blue-400 bg-blue-500/10 border border-blue-400/20 hover:bg-blue-500/20"
				>
					disconnect
				</button>
			{/if}
		</div>
	</div>
</Layout>

{#if apiKey}
	<Realtime
		bind:startConversation
		bind:endConversation
		bind:resetClient
		bind:isConnected
		bind:isRecording
		bind:items
		bind:startRecording
		bind:stopRecording
		{turnDetection}
		{apiKey}
		bind:instructions={contextInstructions}
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

	.calling-animation {
		display: flex;
		gap: 1rem;
		justify-content: center;
		align-items: center;
		margin-bottom: 2rem;
	}

	.calling-circle {
		width: 20px;
		height: 20px;
		background-color: rgb(59, 130, 246);
		border-radius: 50%;
		animation: calling 1.5s infinite ease-in-out;
	}

	.calling-circle:nth-child(1) {
		animation-delay: 0s;
	}

	.calling-circle:nth-child(2) {
		animation-delay: 0.2s;
	}

	.calling-circle:nth-child(3) {
		animation-delay: 0.4s;
	}

	@keyframes calling {
		0%, 100% {
			transform: scale(0.5);
			opacity: 0.3;
		}
		50% {
			transform: scale(1);
			opacity: 1;
		}
	}
</style>
