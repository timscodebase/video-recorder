<script lang="ts">
	type State = 'ready' | 'ready.countdown' | 'recording'
	type Optional<T> = T | undefined
	type Props = {
		width: Optional<number>
		height: Optional<number>
		audio: boolean
		frameRate: number
	}

	let { width, height, audio = true, frameRate = 60 } = $props<Props>()

	let state = $state<State>('ready')
	let recorder: MediaRecorder
	let stream: MediaStream
	let videoChunks: Blob[] = []
	let timerInterval: number
	let timer = $state(3)

	async function startTimer() {
		state = 'ready.countdown'
		return new Promise((resolve) => {
			timerInterval = setInterval(() => {
				timer--
				if (timer === 0) {
					clearInterval(timerInterval)
					resolve(timer)
				}
			}, 1000)
		})
	}

	function downloadVideoRecording(videoChunks: Blob[]) {
		const blob = new Blob(videoChunks, { type: 'video/webm;codecs=vp9' })
		const a = document.createElement('a')
		a.href = URL.createObjectURL(blob)
		a.download = 'video.webm'
		a.click()
	}

	function stopRecording() {
		state = 'ready'
		stream.getTracks().forEach((track) => track.stop())
		recorder && recorder.stop()
		clearInterval(timerInterval)
		timer = 3
	}

	function startRecording() {
		state = 'recording'
		recorder = new MediaRecorder(stream, { mimeType: 'video/webm;codecs=vp9' })
		recorder.start()

		recorder.addEventListener('dataavailable', (e) => {
			videoChunks.push(e.data)
		})

		recorder.addEventListener('stop', () => {
			downloadVideoRecording(videoChunks)
			videoChunks = []
		})
	}

	async function prepareRecording() {
		try {
			stream = await navigator.mediaDevices.getDisplayMedia({
				audio,
				video: { width, height, frameRate },
				// @ts-ignore
				preferCurrentTab: true
			})

			stream.addEventListener('inactive', startRecording)

			await startTimer()
			startRecording()
		} catch (e) {
			console.error(`Error: ${e}`)
			return
		}

		// videoEl.srcObject = mediaStream
	}

	function handleKeydown(event: KeyboardEvent) {
		switch (event.key) {
			case 'R':
				if (state === 'ready') {
					prepareRecording()
				}
				break
			case 'S':
				stopRecording()
				break
		}
	}
</script>

<svelte:window on:keydown={handleKeydown} />

{#if state.includes('ready')}
	<div class="recorder">
		<button on:click|preventDefault={prepareRecording}>
			<div class="circle">
				{#if state === 'ready.countdown'}
					{timer}
				{/if}
			</div>
		</button>

		<div class="info">
			{#if state === 'ready'}
				<div class="shortcut">Shift + R</div>
				<div class="decription">To start recording.</div>
			{/if}
			{#if state === 'recording'}
				<div class="shortcut">Shift + S</div>
				<div class="decription">To stop recording.</div>
			{/if}
		</div>
	</div>
{/if}

<style>
	button {
		background-color: transparent;
		width: 100px;
		height: 100px;
		border: none;
		cursor: pointer;
		padding: 0;
	}

	.circle {
		width: 100%;
		height: 100%;
		background-color: var(--orange);
		border-radius: 50%;
		display: grid;
		place-content: center;
		color: #000;
	}
	.recorder {
		position: absolute;
		top: 40%;
		left: 40%;
		padding: 1rem;
		font-family: 'Manrope Variable', sans-serif;
		color: var(--text);
		background-color: var(--surface);
		border-radius: 4px;
		user-select: none;
		z-index: 10;
	}

	.recorder video {
		width: 100%;
		height: 400px;
		border-radius: 4px;
		aspect-ratio: 16 / 9;
	}

	.recorder .controls {
		display: grid;
		gap: 0.5rem;
	}

	.recorder button {
		font-weight: bold;
		color: hsl(0 0% 98%);
	}

	pre {
		padding: 1rem;
		color: hsl(0 0% 4%);
		background-color: hsl(0 0% 98%);
		border-radius: 4px;
	}
</style>
