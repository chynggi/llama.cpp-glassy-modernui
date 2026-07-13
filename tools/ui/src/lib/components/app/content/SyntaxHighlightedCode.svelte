<script lang="ts">
	import hljs from 'highlight.js';
	import { browser } from '$app/environment';
	import { mode } from 'mode-watcher';

	import githubDarkCss from 'highlight.js/styles/github-dark-dimmed.css?inline';
	import githubLightCss from 'highlight.js/styles/github.css?inline';
	import tokyoNightDarkCss from 'highlight.js/styles/tokyo-night-dark.css?inline';
	import tokyoNightLightCss from 'highlight.js/styles/tokyo-night-light.css?inline';
	import nordCss from 'highlight.js/styles/nord.css?inline';
	import draculaCss from 'highlight.js/styles/base16/dracula.css?inline';
	import gruvboxDarkCss from 'highlight.js/styles/base16/gruvbox-dark-medium.css?inline';
	import gruvboxLightCss from 'highlight.js/styles/base16/gruvbox-light-medium.css?inline';
	import grayscaleCss from 'highlight.js/styles/grayscale.css?inline';
	import { ColorMode } from '$lib/enums';
	import { config } from '$lib/stores/settings.svelte';
	import { SETTINGS_KEYS } from '$lib/constants';

	interface Props {
		code: string;
		language?: string;
		class?: string;
		maxHeight?: string;
		maxWidth?: string;
	}

	let {
		code,
		language = 'text',
		class: className = '',
		maxHeight = '60vh',
		maxWidth = ''
	}: Props = $props();

	let highlightedHtml = $state('');

	function loadHighlightTheme(isDark: boolean, themeStyle: string) {
		if (!browser) return;

		const existingThemes = document.querySelectorAll('style[data-highlight-theme-preview]');
		existingThemes.forEach((style) => style.remove());

		const style = document.createElement('style');
		style.setAttribute('data-highlight-theme-preview', 'true');
		
		let cssContent = isDark ? githubDarkCss : githubLightCss;

		if (themeStyle === 'tokyo-night') {
			cssContent = isDark ? tokyoNightDarkCss : tokyoNightLightCss;
		} else if (themeStyle === 'nord') {
			cssContent = nordCss;
		} else if (themeStyle === 'dracula' || themeStyle === 'synthwave') {
			cssContent = draculaCss;
		} else if (themeStyle === 'gruvbox') {
			cssContent = isDark ? gruvboxDarkCss : gruvboxLightCss;
		} else if (themeStyle === 'monochrome') {
			cssContent = grayscaleCss;
		}

		style.textContent = cssContent;
		document.head.appendChild(style);
	}

	$effect(() => {
		const currentMode = mode.current;
		const isDark = currentMode === ColorMode.DARK;
		const themeStyle = String(config()[SETTINGS_KEYS.THEME_STYLE] ?? 'default');

		loadHighlightTheme(isDark, themeStyle);
	});

	$effect(() => {
		if (!code) {
			highlightedHtml = '';
			return;
		}

		try {
			// Check if the language is supported
			const lang = language.toLowerCase();
			const isSupported = hljs.getLanguage(lang);

			if (isSupported) {
				const result = hljs.highlight(code, { language: lang });
				highlightedHtml = result.value;
			} else {
				// Try auto-detection or fallback to plain text
				const result = hljs.highlightAuto(code);
				highlightedHtml = result.value;
			}
		} catch {
			// Fallback to escaped plain text
			highlightedHtml = code.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
		}
	});
</script>

<div
	class="code-preview-wrapper min-w-0 max-w-full overflow-x-auto rounded-lg border border-border bg-muted {className}"
	style="max-height: {maxHeight}; {maxWidth ? `max-width: ${maxWidth};` : ''}"
>
	<!-- Needs to be formatted as single line for proper rendering -->
	<pre class="m-0"><code class="hljs text-sm leading-relaxed">{@html highlightedHtml}</code></pre>
</div>

<style>
	.code-preview-wrapper pre {
		background: transparent;
	}

	.code-preview-wrapper code {
		background: transparent;
	}
</style>
