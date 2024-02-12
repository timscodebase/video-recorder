<script lang="ts">
	let mediaStream: MediaStream
	let videoEl: HTMLVideoElement
	let settings: MediaTrackSettings

	async function record() {
		mediaStream = await navigator.mediaDevices.getDisplayMedia({
			audio: true,
			video: { width: 1200, height: 800, frameRate: 60 },
			// @ts-ignore
			preferCurrentTab: true
		})

		// videoEl.srcObject = mediaStream
		settings = mediaStream.getVideoTracks()[0].getSettings()

		const mediaRecorder = new MediaRecorder(mediaStream)
		mediaRecorder.addEventListener(`dataavailable`, (e) => {
			videoEl.src = URL.createObjectURL(e.data)
		})
		mediaRecorder.start()
	}

	function stop() {
		mediaStream.getTracks().forEach((track) => track.stop())
	}
</script>

<details>
	<summary>Recorder</summary>

	<!-- svelte-ignore a11y-media-has-caption -->
	<video bind:this={videoEl} controls autoplay />

	<div class="controls">
		<button on:click={record}>Record</button>
		<button on:click={stop}>Stop</button>
	</div>

	{#if settings}
		<pre>{JSON.stringify(settings, null, 2)}</pre>
	{/if}
</details>

<style>
</style>
