<script lang="ts">
	import ChevronDownIcon from '@lucide/svelte/icons/chevron-down';
	import * as Collapsible from '$lib/components/ui/collapsible/index.js';
	import { buttonVariants } from '$lib/components/ui/button/index.js';
	import { Card } from '$lib/components/ui/card';
	import { createAutoScrollController } from '$lib/hooks/use-auto-scroll.svelte';
	import { useThrottle } from '$lib/hooks/use-throttle.svelte';
	import { formatReasoningPreview } from '$lib/utils';
	import { config } from '$lib/stores/settings.svelte';
	import { slide } from 'svelte/transition';
	import type { Snippet } from 'svelte';
	import type { Component } from 'svelte';

	interface Props {
		open?: boolean;
		class?: string;
		icon?: Component;
		iconClass?: string;
		title: string;
		subtitle?: string;
		preview?: string;
		rawContent?: string;
		isStreaming?: boolean;
		onToggle?: () => void;
		children: Snippet;
	}

	let {
		open = $bindable(false),
		class: className = '',
		icon: IconComponent,
		iconClass = 'h-4 w-4',
		title,
		subtitle,
		preview,
		rawContent,
		isStreaming = false,
		onToggle,
		children
	}: Props = $props();

	let contentContainer: HTMLDivElement | undefined = $state();

	const showThoughtInProgress = $derived(config().showThoughtInProgress as boolean);

	let previewKey = useThrottle(() => rawContent ?? preview ?? '', 500);
	let displayedPreview = $state('');
	let displayedOverflow = $state(0);

	// Elapsed thinking timer
	let elapsedSeconds = $state(0);
	let timerInterval: ReturnType<typeof setInterval> | undefined = $state();
	let startTime = $state<number | null>(null);

	$effect(() => {
		if (isStreaming && title.startsWith('Reasoning')) {
			if (!startTime) {
				startTime = Date.now();
				elapsedSeconds = 0;
			}
			if (!timerInterval) {
				timerInterval = setInterval(() => {
					if (startTime) {
						elapsedSeconds = Math.round((Date.now() - startTime) / 100) / 10;
					}
				}, 100);
			}
		} else {
			if (timerInterval) {
				clearInterval(timerInterval);
				timerInterval = undefined;
			}
		}

		return () => {
			if (timerInterval) {
				clearInterval(timerInterval);
			}
		};
	});

	$effect(() => {
		void previewKey.key;
		const content = rawContent ?? preview ?? '';
		const result = formatReasoningPreview(content);
		displayedPreview = result.preview;
		displayedOverflow = result.overflow;
	});

	const autoScroll = createAutoScrollController();

	$effect(() => {
		autoScroll.setContainer(contentContainer);
	});

	$effect(() => {
		// Only auto-scroll when open and streaming
		autoScroll.updateInterval(open && isStreaming);
	});

	function handleScroll() {
		autoScroll.handleScroll();
	}
</script>

<Collapsible.Root
	{open}
	onOpenChange={(value) => {
		open = value;
		onToggle?.();
	}}
	class="{className} my-0!"
>
	<Card
		class="gap-0 border-muted bg-muted/10 py-0 transition-all duration-300 backdrop-blur-md {open ? 'border-accent/40 bg-muted/20 shadow-[0_0_15px_-3px_var(--glow-color)] dark:shadow-[0_0_20px_-3px_var(--glow-color-dark)]' : ''}"
	>
		<Collapsible.Trigger class="flex w-full cursor-pointer items-start justify-between gap-2 p-3">
			<div class="flex min-w-0 items-center gap-2">
				<div class="flex items-center gap-2 text-muted-foreground">
					{#if IconComponent}
						<IconComponent class={iconClass} />
					{/if}

					<span class="font-mono text-sm font-medium">{title}</span>

					{#if subtitle}
						<span class="text-xs italic">{subtitle}</span>
					{:else if title.startsWith('Reasoning') && elapsedSeconds > 0}
						<span class="text-xs italic text-muted-foreground/80">
							{#if isStreaming}
								thinking... ({elapsedSeconds.toFixed(1)}s)
							{:else}
								thought for {elapsedSeconds.toFixed(1)}s
							{/if}
						</span>
					{/if}
				</div>

				{#if displayedPreview && !showThoughtInProgress}
					<div class="flex min-w-0 items-baseline justify-between gap-2">
						<div class="w-3/4 truncate text-xs text-muted-foreground/80">
							{displayedPreview}
						</div>
						{#if displayedOverflow > 0}
							<span class="shrink-0 text-xs text-muted-foreground/60"
								>{displayedOverflow}+ chars</span
							>
						{/if}
					</div>
				{/if}
			</div>

			<div
				class={buttonVariants({
					variant: 'ghost',
					size: 'sm',
					class: 'h-6 w-6 p-0 text-muted-foreground hover:text-foreground'
				})}
			>
				<ChevronDownIcon class="h-4 w-4 transition-transform duration-300 {open ? 'rotate-180' : ''}" />

				<span class="sr-only">Toggle content</span>
			</div>
		</Collapsible.Trigger>

		<Collapsible.Content forceMount>
			{#if open}
				<div
					bind:this={contentContainer}
					transition:slide={{ duration: 250 }}
					class="overflow-y-auto border-t border-muted px-3 pb-3"
					onscroll={handleScroll}
					style="min-height: var(--min-message-height); max-height: var(--max-message-height);"
				>
					{@render children()}
				</div>
			{/if}
		</Collapsible.Content>
	</Card>
</Collapsible.Root>
