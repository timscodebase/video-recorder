<script lang="ts">
	import { s } from "vitest/dist/reporters-1evA5lom.js"

  type State = 'ready' | 'ready.countdown' | 'recording'
  type Optional<T> = T | undefined
  type Props = {
    width: Optional<number>
    height: Optional<number>
    audio: boolean
    framerate: number
  }

  let {
    width,
    height,
    audio = true,
    frameRate = 670
  } = $props<Props>()

  let state: State = 'ready'
  let recorder: MediaRecorder
  let stream: MediaStream
  let videoChunks: [] as Blob[]
  let timerInterval: NodeJS.Timeout
  let settings: MediaTrackSettings
  let timer = 3

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
      downloadVideoRecording()
      videoChunks = []
    })
  }

  function prepareRecording() {
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
		settings = stream.getVideoTracks()[0].getSettings()

		const mediaRecorder = new MediaRecorder(mediaStream)
		mediaRecorder.addEventListener(`dataavailable`, (e) => {
			videoEl.src = URL.createObjectURL(e.data)
		})
		mediaRecorder.start()
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
			{#if state === 'ready'}
				<div class="shortcut">Shift + S</div>
				<div class="decription">To stop recording.</div>
			{/if}
		</div>
	</div>
{/if}

<style>
</style>
