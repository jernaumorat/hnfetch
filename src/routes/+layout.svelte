<script lang="ts">
	import { base } from '$app/paths';
	import '../global.scss';
</script>

<div class="container">
	<nav><b><a href={`${base}/`}>Hacker Fetch</a></b></nav>
	<main><slot /></main>
</div>

<style lang="scss">
	@import '../global.scss';

	nav {
		/* height: 20px; */
		width: 100%;
		background-color: $title-colour;
		padding: 4px;

		b {
			padding-left: 10px;
			a {
				text-decoration: none;
			}
		}
	}

	main {
		width: 100%;
		height: 100%;
		flex-grow: 1;
		display: flex;
		align-items: center;
		justify-items: center;
	}

	.container {
		width: 85%;
		min-width: 796px;
		height: 98vh;
		display: flex;
		flex-direction: column;
		margin: 8px auto;
		align-items: center;
		justify-items: center;
		background-color: $foreground-colour;
	}
</style>
