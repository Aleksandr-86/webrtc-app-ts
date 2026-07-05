<script setup lang="ts">
import { ref } from 'vue'

const captureBtnRef = ref<HTMLButtonElement | null>(null)
const stopBtnRef = ref<HTMLButtonElement | null>(null)
const videoElRef = ref<HTMLVideoElement | null>(null)

const displayMediaOptions: DisplayMediaStreamOptions = {
  audio: true,
  video: {
    frameRate: {
      ideal: 60,
    },
  },
}

let currentStream: MediaStream | null = null

const isStreamActive = ref(false)

function stopScreenCapture() {
  if (currentStream) {
    currentStream.getTracks().forEach((track) => track.stop())
    currentStream = null
  }

  if (videoElRef.value) {
    videoElRef.value.srcObject = null
  }

  isStreamActive.value = false
}

async function initScreenCapture() {
  if (!videoElRef.value) {
    console.error('Элемент #video-el не найден')
    return
  }

  if ('getDisplayMedia' in navigator.mediaDevices) {
    try {
      currentStream = await navigator.mediaDevices.getDisplayMedia(displayMediaOptions)

      videoElRef.value.srcObject = currentStream
      videoElRef.value.play()

      isStreamActive.value = true

      const videoTrack = currentStream.getVideoTracks()[0]

      if (videoTrack) {
        videoTrack.onended = () => stopScreenCapture()
      }
    } catch (err) {
      if (err instanceof Error) {
        if (err.name === 'NotAllowedError') {
          console.warn('Пользователь отменил выбор источника или не дал разрешение.')
        }
      }
    }
  } else {
    console.warn('"getDisplayMedia" не поддерживается данным обозревателем.')
    isStreamActive.value = false
  }
}
</script>

<template>
  <div>
    <video ref="videoElRef"></video>

    <div>
      <button :disabled="isStreamActive" ref="captureBtnRef" @click="initScreenCapture">
        Захватить экран
      </button>
      <button v-if="isStreamActive" ref="stopBtnRef" @click="stopScreenCapture">
        Прекратить захват экрана
      </button>
    </div>
  </div>
</template>

<style scoped>
button {
  cursor: pointer;
  height: fit-content;
}
</style>
