<script lang="ts">
	import { ACCENT_COLORS, SETTINGS_KEYS } from '$lib/constants';
	import { config } from '$lib/stores/settings.svelte';

	const WIDE_CHAT_CLASS = 'wide-chat-mode';

	$effect(() => {
		const currentConfig = config();
		const accentKey = String(currentConfig[SETTINGS_KEYS.ACCENT_COLOR] ?? 'default');
		const accent = ACCENT_COLORS[accentKey] ?? ACCENT_COLORS.default;

		document.documentElement.style.setProperty('--accent-light', accent.light);
		document.documentElement.style.setProperty('--accent-dark', accent.dark);
		document.documentElement.style.setProperty('--glow-color-light', accent.glowLight);
		document.documentElement.style.setProperty('--glow-color-dark', accent.glowDark);

		if (currentConfig[SETTINGS_KEYS.WIDE_CHAT_MODE]) {
			document.documentElement.classList.add(WIDE_CHAT_CLASS);
		} else {
			document.documentElement.classList.remove(WIDE_CHAT_CLASS);
		}
	});
</script>
