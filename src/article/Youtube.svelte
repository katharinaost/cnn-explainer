<script context="module">
	// Load the YouTube IFrame API lazily — only when the user actually activates
	// the player. This keeps page load free of any request to youtube.com
	// (no YouTube tracking cookies until the visitor opts in by clicking).
	let apiLoading = false;
	let apiReady = false;
	const apiReadyCallbacks = [];

	function loadYouTubeApi() {
		return new Promise((resolve) => {
			if (apiReady) { resolve(); return; }
			apiReadyCallbacks.push(resolve);
			if (apiLoading) { return; }
			apiLoading = true;

			window.onYouTubeIframeAPIReady = () => {
				apiReady = true;
				apiReadyCallbacks.splice(0).forEach((cb) => cb());
			};

			let tag = document.createElement("script");
			tag.src = "https://www.youtube.com/iframe_api";
			let firstScriptTag = document.getElementsByTagName("script")[0];
			firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
		});
	}
</script>

<script>
	import { tick } from "svelte";
	export let videoId;
	export let playerId = "player";

	let player;
	let activated = false;     // user has loaded the player
	let pendingStart = null;   // seek target requested before the player is ready

	async function activate(startSecond = 0) {
		pendingStart = startSecond;

		// Player already exists: just seek/play.
		if (player) {
			player.seekTo(startSecond);
			player.playVideo();
			return;
		}

		// Already loading: the pendingStart above will be honored on ready.
		if (activated) { return; }

		activated = true;
		await loadYouTubeApi();
		await tick();              // ensure the <div id={playerId}> is in the DOM
		player = new YT.Player(playerId, {
			videoId,
			events: { onReady: onPlayerReady }
		});
	}

	// Called by the article's timestamp links (currentPlayer.play(seconds)).
	export function play(startSecond = 0) {
		activate(startSecond);
	}

	function onPlayerReady() {
		player.mute();
		if (pendingStart != null) { player.seekTo(pendingStart); }
		player.playVideo();
	}
</script>

{#if activated}
	<div id={playerId} />
{:else}
	<button class="yt-facade" type="button" on:click={() => activate(0)}>
		<span class="yt-facade-play">▶</span>
		<span class="yt-facade-text">
			Load video tutorial
			<small>Loads the player from YouTube when clicked</small>
		</span>
	</button>
{/if}

<style>
	.yt-facade {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 12px;
		width: 100%;
		min-height: 200px;
		padding: 30px;
		border: 1px dashed var(--light-gray, #ccc);
		border-radius: 6px;
		background: rgb(245, 245, 245);
		color: rgb(80, 80, 80);
		cursor: pointer;
		font-size: 16px;
	}

	.yt-facade:hover {
		background: rgb(238, 238, 238);
	}

	.yt-facade-play {
		font-size: 34px;
		line-height: 1;
		color: rgb(200, 50, 50);
	}

	.yt-facade-text {
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: center;
	}

	.yt-facade-text small {
		margin-top: 4px;
		font-size: 12px;
		color: rgb(130, 130, 130);
	}
</style>
