<script lang="ts">
	import { ACCENT_COLORS, SETTINGS_KEYS } from '$lib/constants';
	import { config } from '$lib/stores/settings.svelte';

	const WIDE_CHAT_CLASS = 'wide-chat-mode';
	const FULL_CHAT_CLASS = 'full-chat-mode';

	const THEME_CLASSES = [
		'theme-tokyo-night',
		'theme-nord',
		'theme-dracula',
		'theme-gruvbox',
		'theme-synthwave',
		'theme-soft',
		'theme-monochrome'
	];

	$effect(() => {
		const currentConfig = config();
		const accentKey = String(currentConfig[SETTINGS_KEYS.ACCENT_COLOR] ?? 'default');
		const accent = ACCENT_COLORS[accentKey] ?? ACCENT_COLORS.default;

		document.documentElement.style.setProperty('--accent-light', accent.light);
		document.documentElement.style.setProperty('--accent-dark', accent.dark);
		document.documentElement.style.setProperty('--glow-color-light', accent.glowLight);
		document.documentElement.style.setProperty('--glow-color-dark', accent.glowDark);

		// Handle theme style variations
		const themeStyle = String(currentConfig[SETTINGS_KEYS.THEME_STYLE] ?? 'default');

		THEME_CLASSES.forEach((cls) => {
			document.documentElement.classList.remove(cls);
		});

		if (themeStyle !== 'default') {
			document.documentElement.classList.add(`theme-${themeStyle}`);
		}

		// Handle chat width layout styles
		const chatWidthStyle = String(currentConfig[SETTINGS_KEYS.CHAT_WIDTH_STYLE] ?? 'normal');

		document.documentElement.classList.remove(WIDE_CHAT_CLASS);
		document.documentElement.classList.remove(FULL_CHAT_CLASS);

		if (chatWidthStyle === 'wide') {
			document.documentElement.classList.add(WIDE_CHAT_CLASS);
		} else if (chatWidthStyle === 'full') {
			document.documentElement.classList.add(FULL_CHAT_CLASS);
		}
	});
</script>
