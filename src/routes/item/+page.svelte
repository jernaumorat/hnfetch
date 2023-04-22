<script lang="ts">
	import { onMount } from 'svelte';
	import { domToPng } from 'modern-screenshot';

	type PageData = Partial<{
		id: string;
		type: string;
		time: string;
		by: string;
		text?: string;
		story: string;
		storyId: string;
		loading: boolean;
	}>;
	let data: PageData = {};
	let loading = true;
	let error = 0;
	let errorMsg = '';

	const getStoryTitle = async (id: string): Promise<[string, string]> => {
		const res = await fetch(`https://hacker-news.firebaseio.com/v0/item/${id}.json`);

		const { type, parent, title } = await res.json();
		if (type === 'story') {
			return [title, id];
		}

		return getStoryTitle(parent);
	};

	const formatters = [
		new Intl.DateTimeFormat('en-US', { timeZone: 'UTC' }),
		new Intl.DateTimeFormat('en-US', { dateStyle: 'long', timeZone: 'UTC' }),
		new Intl.DateTimeFormat('en-US', { dateStyle: 'long', timeStyle: 'short', timeZone: 'UTC' }),
		new Intl.DateTimeFormat('en-GB', { timeZone: 'UTC' }),
		new Intl.DateTimeFormat('en-GB', { dateStyle: 'long', timeZone: 'UTC' }),
		new Intl.DateTimeFormat('en-GB', { dateStyle: 'long', timeStyle: 'short', timeZone: 'UTC' }),
		{ format: (d: number) => new Date(d).toUTCString() },
		{ format: (d: number) => new Date(d).toISOString() }
	];
	let formatIndex = 0;

	const saveImage = () => {
		const bubble = document.getElementById('bubble');
		if (!bubble) {
			console.log('no bubble');
			return;
		}
		const downloadLink = document.createElement('a');
		downloadLink.setAttribute('download', `hnfetch_${data.id}.png`);
		domToPng(bubble, { scale: 2 }).then((dlUrl) => {
			downloadLink.setAttribute('href', dlUrl);
			downloadLink.click();
		});
		downloadLink.remove();
	};

	onMount(async () => {
		const id = new URLSearchParams(window.location.search).get('id');

		if (!id) {
			error = 400;
			errorMsg = 'Bad request';
			loading = false;
			return;
		}

		const res = await fetch(`https://hacker-news.firebaseio.com/v0/item/${id}.json`);

		if (!res.ok) {
			error = res.status;
			errorMsg = res.statusText;
			loading = false;
			return;
		}

		const json = await res.json();
		if (!json) {
			error = 400;
			errorMsg = 'Item not found';
			loading = false;
			return;
		}

		const { type, time, by, text, parent } = json;

		if (!text) {
			error = 415;
			errorMsg = 'Item must have body text (comment or text post)';
			loading = false;
			return;
		}

		const [story, storyId] = await getStoryTitle(parent);

		data = { id, type, time, by, text, story, storyId };
		loading = false;
	});
</script>

{#if loading}
	<span id="loading">loading...</span>
{:else if error}
	<span id="error"
		>{error}
		<p>{errorMsg}</p></span
	>
{:else}
	<div id="bubble">
		<div class="item">
			<div class="item__header">
				<span
					class="link by"
					on:click={() => window.open(`https://news.ycombinator.com/user?id=${data.by}`, '_blank')}
					on:keydown={() =>
						window.open(`https://news.ycombinator.com/user?id=${data.by}`, '_blank')}
					>{data.by}</span
				>
				<span
					class="link date"
					on:click={() => (formatIndex = (formatIndex + 1) % formatters.length)}
					on:keydown={() => (formatIndex = (formatIndex + 1) % formatters.length)}
				>
					{formatters[formatIndex].format(parseInt(data.time ?? '0') * 1000)}
				</span>
			</div>
			<div class="item__body"><p class="item__body__text">{@html data.text}</p></div>
			<div class="item__footer">
				<span
					class="link story"
					on:click={() => window.open(`https://news.ycombinator.com/item?id=${data.id}`, '_blank')}
					on:keydown={() =>
						window.open(`https://news.ycombinator.com/item?id=${data.id}`, '_blank')}
					>{data.story}</span
				>
			</div>
		</div>
	</div>

	<div class="controls">
		<button on:click={saveImage}>Save Image</button>
	</div>
{/if}

<style lang="scss">
	@import '../../global.scss';

	#bubble {
		background-color: $background-colour;
		border-radius: 12px;
		width: 85%;
		max-width: 800px;
		margin: 0 auto;
		padding: 3em;
	}

	.item {
		background-color: $foreground-colour;
		border-radius: 12px;
		margin: 0 auto;
		padding: 1.5em;
		box-shadow: $item-shadow;

		&__header,
		&__footer {
			color: $label-text-colour;
			display: flex;
			justify-content: space-between;

			> span {
				font-size: 8pt;
			}
		}

		&__header {
			margin-bottom: 0.5em;
		}

		&__footer {
			margin-top: 0.5em;
		}

		&__body {
			text-align: justify;
			line-height: 1.25em;

			&__text {
				/* display: flex; */
				flex-direction: column;
				gap: 8px;
			}
		}
	}

	:global(.item__body__text > p) {
		margin-top: 8px;
	}
	p,
	span {
		user-select: none;
	}

	#loading {
		margin: 0 auto;
		text-align: center;
		color: $label-text-colour;
	}

	#error {
		margin: 0 auto;
		text-align: center;
		color: $label-text-colour;
	}

	.link {
		cursor: pointer;
	}

	.by {
		text-align: left;
		flex: 2 0 0;
	}

	.story {
		text-align: center;
		flex: 3 0 0;
	}

	.date {
		text-align: right;
		flex: 2 0 0;
	}
</style>
