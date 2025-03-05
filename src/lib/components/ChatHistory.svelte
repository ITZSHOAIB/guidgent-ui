<script lang="ts">
	import Icon from '@iconify/svelte';
	import { marked } from 'marked';
	import { onMount, tick } from 'svelte';
	import { welcomeMessage } from '$lib/welcome';
	import chatEmpty from '$lib/assets/chat_empty2.svg';

	const { chatHistory: chatHistoryProp, isStreaming } = $props<{
		chatHistory: { text?: string; role: 'user' | 'agent' }[];
		isStreaming: boolean;
	}>();

	let chatHistory = $state(chatHistoryProp);

	let chatContainer: HTMLDivElement;

	const scrollToBottom = async (node: HTMLDivElement) => {
		node.scroll({ top: node.scrollHeight, behavior: 'smooth' });
	};

	onMount(() => {
		scrollToBottom(chatContainer);
	});

	$effect(() => {
		chatHistory = [...chatHistoryProp];

		const autoscroll =
			chatContainer &&
			chatContainer.offsetHeight + chatContainer.scrollTop > chatContainer.scrollHeight - 50;
		console.log(autoscroll);
		if (autoscroll) {
			tick().then(() => scrollToBottom(chatContainer));
		}
	});
</script>

<div
	bind:this={chatContainer}
	class="flex w-full max-w-4xl flex-1 flex-col gap-4 overflow-y-auto p-4 text-lg"
>
	{#if !chatHistory || chatHistory.length === 0}
		<div class="flex h-96 flex-1 items-center justify-center">
			<div class="flex h-full flex-col items-center gap-4 p-8 text-center">
				<img src={chatEmpty} alt="Maths Guide" class="max-h-1/2 max-w-full" />
				<div class="prose prose-lg text-left">{@html marked.parse(welcomeMessage)}</div>
			</div>
		</div>
	{/if}

	{#each chatHistory as chat, index}
		{#if chat.role === 'user'}
			<div class="bg-base-300 self-end rounded-2xl rounded-tr-none p-3 px-4">
				{chat.text}
			</div>
		{/if}
		{#if chat.role === 'agent'}
			<div class="flex flex-col gap-2 py-4 pr-2 md:flex-row">
				<Icon icon="mingcute:ai-fill" class="h-6 w-6" />
				<div class="prose prose-lg dark:text-base-content w-full flex-1">
					{@html marked(chat.text!)}
					{#if isStreaming && index === chatHistory.length - 1}
						<span class="loading loading-dots loading-lg"></span>
					{/if}
				</div>
			</div>
		{/if}
	{/each}
</div>
