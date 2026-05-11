<script lang="ts">
	import Icon from '@iconify/svelte';
	import { Button } from '$lib/components/ui/button';
	import * as Dialog from '$lib/components/ui/dialog';
	import { fetchOutputList, getImageUrl, getImageProxyUrl } from '$lib/draw/api/client';
	import type { DrawOutputItem } from '$lib/draw/types';

	let items = $state<DrawOutputItem[]>([]);
	let total = $state(0);
	let loading = $state(false);
	let offset = $state(0);
	const limit = 30;
	let hasMore = $derived(offset < total);

	let lightboxOpen = $state(false);
	let lightboxIndex = $state(0);

	$effect(() => {
		loadGallery();
	});

	async function loadGallery() {
		loading = true;
		try {
			const res = await fetchOutputList(limit, 0);
			items = res.items;
			total = res.total;
			offset = res.items.length;
		} catch {
			items = [];
		} finally {
			loading = false;
		}
	}

	async function loadMore() {
		if (loading || !hasMore) return;
		loading = true;
		try {
			const res = await fetchOutputList(limit, offset);
			items = [...items, ...res.items];
			offset += res.items.length;
		} catch {
			// ignore
		} finally {
			loading = false;
		}
	}

	function openLightbox(index: number) {
		lightboxIndex = index;
		lightboxOpen = true;
	}

	function prev() {
		lightboxIndex = (lightboxIndex - 1 + items.length) % items.length;
	}

	function next() {
		lightboxIndex = (lightboxIndex + 1) % items.length;
	}

	const currentItem = $derived(items[lightboxIndex]);
</script>

<div class="space-y-3">
	<div class="flex items-center justify-between">
		<h3 class="text-sm font-medium flex items-center gap-1.5">
			<Icon icon="mdi:image-multiple-outline" class="size-4" />
			画廊
			<span class="text-xs text-muted-foreground">({items.length}/{total})</span>
		</h3>
		<Button variant="ghost" size="sm" onclick={loadGallery} disabled={loading}>
			<Icon icon="mdi:refresh" class="size-4" />
		</Button>
	</div>

	{#if items.length === 0 && !loading}
		<div class="text-xs text-muted-foreground py-8 text-center">暂无图片</div>
	{:else}
		<div class="grid grid-cols-3 sm:grid-cols-4 md:grid-cols-5 gap-1.5">
			{#each items as item, i}
				<button
					class="aspect-square rounded-md overflow-hidden border hover:ring-2 hover:ring-primary/50 transition-all"
					onclick={() => openLightbox(i)}
				>
					<img
						src={getImageProxyUrl(item.path)}
						alt={item.path}
						class="w-full h-full object-cover"
						loading="lazy"
					/>
				</button>
			{/each}
		</div>

		{#if hasMore}
			<div class="text-center">
				<Button variant="outline" size="sm" onclick={loadMore} disabled={loading}>
					{#if loading}
						<Icon icon="mdi:loading" class="size-4 animate-spin mr-1" />
					{/if}
					加载更多
				</Button>
			</div>
		{/if}
	{/if}
</div>

<Dialog.Root bind:open={lightboxOpen}>
	<Dialog.Content class="max-w-3xl p-0 overflow-hidden">
		{#if currentItem}
			<div class="relative">
				<img
					src={getImageUrl(currentItem.path)}
					alt={currentItem.path}
					class="w-full max-h-[70vh] object-contain"
					loading="lazy"
				/>
				<div class="absolute bottom-0 inset-x-0 bg-gradient-to-t from-black/60 to-transparent p-3">
					<div class="flex items-center justify-between text-white text-xs">
						<span class="truncate">{currentItem.path}</span>
						<a
							href={getImageUrl(currentItem.path, true)}
							download
							class="p-1.5 rounded bg-white/20 hover:bg-white/30"
						>
							<Icon icon="mdi:download" class="size-4" />
						</a>
					</div>
				</div>
			</div>
			{#if items.length > 1}
				<div class="flex items-center justify-between px-3 py-2 border-t">
					<Button variant="ghost" size="sm" onclick={prev}>
						<Icon icon="mdi:chevron-left" class="size-5" />
					</Button>
					<span class="text-xs text-muted-foreground">{lightboxIndex + 1} / {items.length}</span>
					<Button variant="ghost" size="sm" onclick={next}>
						<Icon icon="mdi:chevron-right" class="size-5" />
					</Button>
				</div>
			{/if}
		{/if}
	</Dialog.Content>
</Dialog.Root>
