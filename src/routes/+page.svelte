<script>
// @ts-nocheck

	import welcome from '$lib/images/svelte-welcome.webp';
	import welcome_fallback from '$lib/images/svelte-welcome.png';
	import {fetchFile, createFFmpeg} from '@ffmpeg/ffmpeg'
	import { onMount } from 'svelte';

	const ffmpeg = createFFmpeg({log: true});

	let ready = false;
	/**
	 * @type {string | null}
	 */
	$: file = null;
	$: gif = null;


	async function convert() {
		ffmpeg.FS('writeFile', 'test.mp4', await fetchFile(file));
		
		await ffmpeg.run('-i', 'test.mp4', '-t', '2.5', '-ss', '2.0', '-f', 'gif', 'out.gif');
		const data = ffmpeg.FS('readFile', 'out.gif');
		const url = URL.createObjectURL(new Blob([data.buffer], {type: 'image/gif'}))

		gif = url
	}
	function getFile(event) {
		file = event.target.files.item(0)
		console.log(file)
	}


	async function load() {
		await ffmpeg.load();
		ready = true;
	}

	onMount (async () => {
		await load();
	});

</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Video to GIF" />
</svelte:head>

<section>
	<h1>
		<span class="welcome">
			<picture>
				<source srcset={welcome} type="image/webp" />
				<img src={welcome_fallback} alt="Welcome" />
			</picture>
		</span>

		to your new<br />SvelteKit app
	</h1>

	{#if ready}
		<input type="file" on:change={getFile}/>
		{#if file}
			<video src={URL.createObjectURL(file)} >
				<track kind="captions" />
			</video>
		{/if}

		{#if gif}
			<img src={gif} />
		{/if}
		<button
			on:click={convert}
			disabled={!file}
		>Convert</button>
	{:else}
		<p>loading...</p>
		
	{/if}
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		width: 100%;
	}

	.welcome {
		display: block;
		position: relative;
		width: 100%;
		height: 0;
		padding: 0 0 calc(100% * 495 / 2048) 0;
	}

	.welcome img {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		display: block;
	}
	video{
		width: 200px;
	}
</style>
