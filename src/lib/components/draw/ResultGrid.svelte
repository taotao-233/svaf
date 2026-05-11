<script lang="ts">
	import Icon from '@iconify/svelte';
	import * as Dialog from '$lib/components/ui/dialog';
	import { Button } from '$lib/components/ui/button';
	import { getImageProxyUrl, getImageUrl } from '$lib/draw/api/client';

	let {
		images = []
	}: {
		images?: { url: string; filename: string }[];
	} = $props();

	let lightboxOpen = $state(false);
	let lightboxIndex = $state(0);

	function openLightbox(index: number) {
		lightboxIndex = index;
		lightboxOpen = true;
	}

	function prev() {
		lightboxIndex = (lightboxIndex - 1 + images.length) % images.length;
	}

	function next() {
		lightboxIndex = (lightboxIndex + 1) % images.length;
	}

	function download() {
		const img = images[lightboxIndex];
		if (!img) return;
		const a = document.createElement('a');
		a.href = getImageUrl(img.filename, true);
		a.download = img.filename;
		a.click();
	}

	const currentImage = $derived(images[lightboxIndex]);
</script>

{#if images.length > 0}
	<div class="space-y-2">
		<h3 class="text-sm font-medium flex items-center gap-1.5">
			<Icon icon="mdi:image-multiple-outline" class="size-4" />
			生成结果
			<span class="text-xs text-muted-foreground">({images.length})</span>
		</h3>
		<div class="grid grid-cols-2 md:grid-cols-3 gap-2">
			{#each images as img, i}
				<button
					class="aspect-square rounded-lg overflow-hidden border hover:ring-2 hover:ring-primary/50 transition-all"
					onclick={() => openLightbox(i)}
				>
					<img
						src={getImageProxyUrl(img.filename)}
						alt={img.filename}
						class="w-full h-full object-cover"
						loading="lazy"
					/>
				</button>
			{/each}
		</div>
	</div>
{/if}

<Dialog.Root bind:open={lightboxOpen}>
	<Dialog.Content class="max-w-3xl p-0 overflow-hidden">
		{#if currentImage}
			<div class="relative">
				<img
					src={getImageUrl(currentImage.filename)}
					alt={currentImage.filename}
					class="w-full max-h-[70vh] object-contain"
					loading="lazy"
				/>
				<div class="absolute bottom-0 inset-x-0 bg-gradient-to-t from-black/60 to-transparent p-3">
					<div class="flex items-center justify-between text-white text-xs">
						<span class="truncate">{currentImage.filename}</span>
						<div class="flex gap-1">
							<Button size="sm" variant="secondary" onclick={download}>
								<Icon icon="mdi:download" class="size-4" />
							</Button>
						</div>
					</div>
				</div>
			</div>
			{#if images.length > 1}
				<div class="flex items-center justify-between px-3 py-2 border-t">
					<Button variant="ghost" size="sm" onclick={prev}>
						<Icon icon="mdi:chevron-left" class="size-5" />
					</Button>
					<span class="text-xs text-muted-foreground">{lightboxIndex + 1} / {images.length}</span>
					<Button variant="ghost" size="sm" onclick={next}>
						<Icon icon="mdi:chevron-right" class="size-5" />
					</Button>
				</div>
			{/if}
		{/if}
	</Dialog.Content>
</Dialog.Root>
