<template>
  <div
      class="!overflow-x-hidden !overflow-y-hidden h-screen w-screen bg-gradient-to-b from-indigo-500 via-purple-500 to-pink-500 flex justify-center items-center relative"
      @touchstart="handleTouchStart"
      @touchend="handleTouchEnd"
  >
    <div :class="`${activeTouches.length ? 'opacity-0' : 'opacity-100'} transition duration-500 h-1/2 flex items-center flex-col text-white space-y-2`">
      <p class="text-5xl font-bold">CHOOSER 2</p>
      <p class="font-light">Un sélecteur de doigts aléatoire.</p>
    </div>
    <div
        v-for="(touch, index) in activeTouches"
        :key="touch.identifier"
        :class="[
        'absolute rounded-full flex justify-center items-center select-none transition-all duration-500',
        selectedTouchId === touch.identifier
          ? 'w-40 h-40 bg-white border-8 border-white'
          : selectedTouchId
          ? 'hidden'
          : 'w-24 h-24 border-8 border-white pulse'
      ]"
        :style="{
        top: `${touch.clientY - (selectedTouchId === touch.identifier ? 80 : 48)}px`,
        left: `${touch.clientX - (selectedTouchId === touch.identifier ? 80 : 48)}px`,
      }"
    ></div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface TouchInfo {
  identifier: number
  clientX: number
  clientY: number
}

const activeTouches = ref<TouchInfo[]>([])
const selectedTouchId = ref<number | null>(null)
const countdown = ref<number>(0)
let countdownTimer: number | null = null
let secondTouchId: number | null = null

const handleTouchStart = (event: TouchEvent) => {
  for (let i = 0; i < event.changedTouches.length; i++) {
    const touch = event.changedTouches[i]
    activeTouches.value.push({
      identifier: touch.identifier,
      clientX: touch.clientX,
      clientY: touch.clientY,
    })
    if (activeTouches.value.length === 2) {
      secondTouchId = touch.identifier
      resetCountdown()
    }
  }
}

const handleTouchEnd = (event: TouchEvent) => {
  for (let i = 0; i < event.changedTouches.length; i++) {
    const touch = event.changedTouches[i]
    const index = activeTouches.value.findIndex(
        (t) => t.identifier === touch.identifier
    )
    if (index !== -1) {
      activeTouches.value.splice(index, 1)
    }
    // Réinitialiser la partie si le doigt sélectionné est retiré
    if (touch.identifier === selectedTouchId.value) {
      resetGame()
    }
  }
  if (activeTouches.value.length < 2) {
    clearCountdown()
  }
}

const resetCountdown = () => {
  clearCountdown()
  countdown.value = 3
  countdownTimer = window.setInterval(() => {
    if (countdown.value > 0) {
      countdown.value--
    } else {
      chooseSecondTouch()
      clearCountdown()
    }
  }, 1000)
}

const clearCountdown = () => {
  if (countdownTimer !== null) {
    clearInterval(countdownTimer)
    countdownTimer = null
  }
  countdown.value = 0
}

const chooseSecondTouch = () => {
  if (secondTouchId !== null) {
    selectedTouchId.value = secondTouchId
  }
}

const resetGame = () => {
  selectedTouchId.value = null
  activeTouches.value = []
  secondTouchId = null
  clearCountdown()
}
</script>

<style scoped>
@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
  }
}

.pulse {
  animation: pulse 1s infinite;
}
</style>
