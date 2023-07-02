<!-- YOU CAN DELETE EVERYTHING IN THIS PAGE -->
<script lang="ts">
    import openai from "$lib/openai";
    import {Avatar} from "@skeletonlabs/skeleton";
    import moment from "moment";

    type MessageFeed = {
        id: number;
        host: boolean;
        timestamp: string;
        message: string;
    };

    let elemChat: HTMLElement;
    let messageFeeds: MessageFeed[] = [];
    let currentMessage = '';
    let disableSend = false;
    $: disableSend = currentMessage.length === 0;

    function scrollChatBottom(): void {
        elemChat.scrollTo({ top: elemChat.scrollHeight, behavior: 'smooth' });
    }

    async function handleSend() {
        disableSend = true;

        const newMessage = {
            id: messageFeeds.length,
            host: true,
            timestamp: moment(new Date()).format('YYYY/MM/DD HH:mm:ss'),
            message: currentMessage,
        };

        messageFeeds = [...messageFeeds, newMessage];
        currentMessage = '';

        // Smoothly scroll to the bottom of the feed
        setTimeout(() => { scrollChatBottom(); }, 0);

        const res = await openai.createChatCompletion({
            model: "gpt-3.5-turbo",
            messages: messageFeeds.map(m => { return { role: "user", content: m.message }}),
        });

        const resMessage = {
            id: messageFeeds.length,
            host: false,
            timestamp: moment(new Date()).format('YYYY/MM/DD HH:mm:ss'),
            message: res.status === 200 ? res.data.choices[0].message.content : 'Error...',
        };

        messageFeeds = [...messageFeeds, resMessage];

        // Smoothly scroll to the bottom of the feed
        setTimeout(() => { scrollChatBottom(); }, 0);
        disableSend = false;
    }
</script>

<div bind:this={elemChat} class="h-full grid grid-rows-[1fr_auto] gap-1">
    <div class="bg-surface-500/30 p-4 overflow-y-auto mb-16">
        <section class="w-full h-full space-y-4">
            {#each messageFeeds as bubble}
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
            {#if  messageFeeds.length > 0 && disableSend}
                <!-- OpenAI Message Bubble Placeholder -->
                <div class="grid grid-cols-[auto_1fr] gap-2">
                    <Avatar initials="AI" width="w-12" />
                    <div class="card p-4 variant-soft rounded-tl-none space-y-2">
                        <header class="flex justify-between items-center">
                            <p class="font-bold">OpenAI</p>
                        </header>
                        <div class="placeholder animate-pulse" />
                    </div>
                </div>
            {/if}
        </section>
    </div>
    <div class="bg-surface-500/30 p-4 w-full absolute bottom-0">
        <div class="input-group input-group-divider grid-cols-[auto_1fr_auto] rounded-container-token">
            <button class="input-group-shim">+</button>
            <input
                bind:value={currentMessage}
                class="bg-transparent border-0 ring-0"
                type="text"
                name="prompt"
                id="prompt"
                placeholder="Write a message..."/>
            <button class="variant-filled-primary" on:click={handleSend} disabled="{disableSend}">Send</button>
        </div>
    </div>
</div>

<style lang="postcss">
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
