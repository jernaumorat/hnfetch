<script lang="ts">
	import { goto } from '$app/navigation';
	import { base } from '$app/paths';

	let invalid = false;

	const idRegex = /(\d)+$/;

	const validate = (text: string) => {
		invalid = false;

		const idMatch = idRegex.exec(text);

		if (!idMatch) {
			invalid = true;
			return;
		}

		goto(`${base}/item?id=` + idMatch[0]);
	};

	const handlePaste = (event: ClipboardEvent) => {
		const pasteText = event.clipboardData?.getData('text');
		if (!pasteText) {
			return;
		}

		return validate(pasteText);
	};
	const handleKeypress = (event: KeyboardEvent) => {
		if (event.key === 'Enter') {
			//@ts-expect-error
			validate(event.target.value);
		}
	};
</script>

<input
	class={invalid ? 'invalid' : ''}
	on:paste={handlePaste}
	on:keydown={handleKeypress}
	on:blur={() => (invalid = false)}
	placeholder="Enter a comment ID or link..."
/>

<style lang="scss">
	@import '../global.scss';

	input {
		width: 85%;
		max-width: 500px;
		height: 2em;
		padding: 1.5em;
		margin: 0 auto;
		border: 1px $label-text-colour solid;
		border-radius: 4px;

		&:focus {
			outline: none;
		}

		&::placeholder {
			color: $label-text-colour;
		}
	}

	.invalid {
		border: 1px red solid;
	}
</style>
