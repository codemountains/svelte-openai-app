<!-- YOU CAN DELETE EVERYTHING IN THIS PAGE -->
<script lang="ts">
	import openai from "$lib/openai";
	import {Avatar} from "@skeletonlabs/skeleton";

	let elemChat: HTMLElement;
	let messageFeeds = [];
	let currentMessage = '';

	function scrollChatBottom(): void {
		elemChat.scrollTo({ top: elemChat.scrollHeight, behavior: 'smooth' });
	}

	async function handleSend() {
		const newMessage = {
			id: messageFeeds.length,
			host: true,
			timestamp: new Date(),
			message: currentMessage,
		};

		messageFeeds = [...messageFeeds, newMessage];
		currentMessage = '';

		// Smoothly scroll to the bottom of the feed
		setTimeout(() => { scrollChatBottom(); }, 0);

		const res = await openai.createChatCompletion({
			model: "gpt-3.5-turbo",
			messages: messageFeeds.map(m => { return { role: "user", content: m.message as string }}),
		});

		if (res.status === 200) {
			const resMessage = {
				id: messageFeeds.length,
				host: false,
				timestamp: new Date(),
				message: res.data.choices[0].message.content,
			};

			messageFeeds = [...messageFeeds, resMessage];
		} else {
			const resMessage = {
				id: messageFeeds.length,
				host: false,
				timestamp: new Date(),
				message: 'Error...',
			};

			messageFeeds = [...messageFeeds, resMessage];
		}

		// Smoothly scroll to the bottom of the feed
		setTimeout(() => { scrollChatBottom(); }, 0);
	}
</script>

<div bind:this={elemChat} class="h-full grid grid-rows-[1fr_auto] gap-1">
	<div class="bg-surface-500/30 p-4 overflow-y-auto mb-16">
		<section class="w-full h-full space-y-4">
			{#each messageFeeds as bubble, i}
				{#if bubble.host === true}
					<!-- Your Message Bubble -->
					<div class="grid grid-cols-[1fr_auto] gap-2">
						<div class="card p-4 rounded-tr-none space-y-2 variant-soft-primary">
							<header class="flex justify-between items-center">
								<p class="font-bold">You</p>
								<small class="opacity-50">{bubble.timestamp}</small>
							</header>
							<p>{bubble.message}</p>
						</div>
						<Avatar initials="Y" width="w-12" />
					</div>
				{:else}
					<!-- OpenAI Message Bubble -->
					<div class="grid grid-cols-[auto_1fr] gap-2">
						<Avatar initials="AI" width="w-12" />
						<div class="card p-4 variant-soft rounded-tl-none space-y-2">
							<header class="flex justify-between items-center">
								<p class="font-bold">OpenAI</p>
								<small class="opacity-50">{bubble.timestamp}</small>
							</header>
							<p>{bubble.message}</p>
						</div>
					</div>
				{/if}
			{/each}
		</section>
	</div>
	<div class="bg-surface-500/30 p-4 w-full absolute bottom-0">
		<div class="input-group input-group-divider grid-cols-[auto_1fr_auto] rounded-container-token">
			<button class="input-group-shim">+</button>
			<textarea
				bind:value={currentMessage}
				class="bg-transparent border-0 ring-0"
				name="prompt"
				id="prompt"
				placeholder="Write a message..."
				rows="1"/>
			<button class="variant-filled-primary" on:click={handleSend}>Send</button>
		</div>
	</div>
</div>

<style lang="postcss">
	figure {
		@apply flex relative flex-col;
	}
	@keyframes glow {
		0% {
			@apply bg-primary-400/50;
		}
		33% {
			@apply bg-secondary-400/50;
		}
		66% {
			@apply bg-tertiary-400/50;
		}
		100% {
			@apply bg-primary-400/50;
		}
	}
	@keyframes pulse {
		50% {
			transform: scale(1.5);
		}
	}
</style>