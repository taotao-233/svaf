<script lang="ts">
	import Icon from '@iconify/svelte';
	import { Badge } from '$lib/components/ui/badge';
	import type { WsRunMessage } from '$lib/draw/types';

	let {
		messages = [],
		visible = false,
		busy = false
	}: {
		messages?: WsRunMessage[];
		visible?: boolean;
		busy?: boolean;
	} = $props();

	let llmText = $state('');
	let llmVisible = $state(false);
	let progressNode = $state('');
	let progressValue = $state(0);
	let progressMax = $state(0);
	let progressDone = $state(0);
	let progressTotal = $state(0);
	let progressText = $state('');
	let logLines = $state<string[]>([]);
	let previewSrc = $state('');
	let lastProcessed = $state(0);

	$effect(() => {
		if (messages.length <= lastProcessed) return;
		for (let i = lastProcessed; i < messages.length; i++) {
			const msg = messages[i];
			switch (msg.type) {
				case 'log':
					logLines = [...logLines, msg.message];
					break;
				case 'llm_start':
					llmVisible = true;
					llmText = '';
					break;
				case 'llm_chunk':
					llmText += msg.delta;
					break;
				case 'llm_done':
					llmVisible = false;
					break;
				case 'progress':
					progressNode = msg.node;
					progressValue = msg.value;
					progressMax = msg.max;
					progressDone = msg.done;
					progressTotal = msg.total;
					if (msg.max > 1) {
						progressText = `${msg.node} ${msg.value}/${msg.max} (${Math.round((msg.value / msg.max) * 100)}%) 节点 ${msg.done}/${msg.total}`;
					} else {
						progressText = msg.node;
					}
					break;
				case 'prompt_id':
					progressText = '生成中...';
					break;
				case 'preview':
					previewSrc = msg.image;
					break;
				case 'done':
					progressText = '完成';
					break;
				case 'error':
					progressText = `失败：${msg.message}`;
					break;
			}
		}
		lastProcessed = messages.length;
	});

	export function reset() {
		llmText = '';
		llmVisible = false;
		progressNode = '';
		progressValue = 0;
		progressMax = 0;
		progressDone = 0;
		progressTotal = 0;
		progressText = '';
		logLines = [];
		previewSrc = '';
		lastProcessed = 0;
	}

	const progressPercent = $derived(progressMax > 0 ? Math.round((progressValue / progressMax) * 100) : 0);
</script>

{#if visible}
	<div class="space-y-3">
		<!-- Preview image -->
		{#if previewSrc}
			<div class="rounded-lg overflow-hidden border">
				<img src={previewSrc} alt="预览" class="w-full max-h-64 object-contain" loading="lazy" />
			</div>
		{/if}

		<!-- Progress bar -->
		<div class="space-y-1.5">
			<div class="flex items-center justify-between text-xs">
				<span class="font-medium flex items-center gap-1.5">
					{#if busy}
						<Icon icon="mdi:loading" class="size-3.5 animate-spin" />
					{:else}
						<Icon icon="mdi:check-circle-outline" class="size-3.5 text-green-500" />
					{/if}
					{progressText || '连接中...'}
				</span>
				{#if progressMax > 1}
					<Badge variant="secondary" class="text-[10px]">{progressPercent}%</Badge>
				{/if}
			</div>
			<div class="w-full h-2 bg-muted rounded-full overflow-hidden">
				<div
					class="h-full bg-primary rounded-full transition-all duration-300"
					style="width: {progressPercent}%"
				></div>
			</div>
		</div>

		<!-- LLM streaming -->
		{#if llmVisible || llmText}
			<div class="space-y-1">
				<div class="text-xs font-medium flex items-center gap-1.5">
					<Icon icon="mdi:brain" class="size-3.5" />
					LLM 处理中...
					{#if llmVisible}
						<Icon icon="mdi:loading" class="size-3 animate-spin" />
					{/if}
				</div>
				<pre class="text-xs bg-yellow-50 dark:bg-yellow-950/30 border rounded-md p-2 max-h-40 overflow-y-auto whitespace-pre-wrap">{llmText}</pre>
			</div>
		{/if}

		<!-- Log area (collapsed) -->
		{#if logLines.length > 0}
			<details class="text-xs">
				<summary class="cursor-pointer text-muted-foreground hover:text-foreground">
					日志 ({logLines.length})
				</summary>
				<pre class="mt-1 bg-muted rounded-md p-2 max-h-32 overflow-y-auto whitespace-pre-wrap">{logLines.join('\n')}</pre>
			</details>
		{/if}
	</div>
{/if}

<style>
	summary::-webkit-details-marker {
		display: none;
	}
</style>
