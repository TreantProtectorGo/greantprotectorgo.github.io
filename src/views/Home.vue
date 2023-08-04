<template>
  <div>
    <p class="error">{{ error }}</p>

    <p class="decode-result">
      Result: <b>{{ result }}</b>
    </p>

    <qrcode-capture @detect="onDetect" ></qrcode-capture>
    <qrcode-stream @detect="onDetect" :track="paintOutline" @error="onError" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

import { QrcodeStream, QrcodeCapture } from 'vue-qrcode-reader'

// data
const result = ref('')
const error = ref('')

// methods
const onDetect = detectedCodes => {
  console.log(detectedCodes)

  const [ firstCode ] = detectedCodes
  result.value = firstCode.rawValue
  //window.location.href = result.value;
}

const paintOutline = (detectedCodes, ctx) => {
  for (const detectedCode of detectedCodes) {
    const {
      boundingBox: { x, y, width, height }
    } = detectedCode

    ctx.lineWidth = 2
    ctx.strokeStyle = '#007bff'
    ctx.strokeRect(x, y, width, height)
  }
}
//vite-vue3-tailwind-starter
const onError = err => {
  error.value = `[${err.name}]: `

  if (err.name === 'NotAllowedError') {
    error.value += 'you need to grant camera access permission'
  } else if (err.name === 'NotFoundError') {
    error.value += 'no camera on this device'
  } else if (err.name === 'NotSupportedError') {
    error.value += 'secure context required (HTTPS, localhost)'
  } else if (err.name === 'NotReadableError') {
    error.value += 'is the camera already in use?'
  } else if (err.name === 'OverconstrainedError') {
    error.value += 'installed cameras are not suitable'
  } else if (err.name === 'StreamApiNotSupportedError') {
    error.value += 'Stream API is not supported in this browser'
  } else if (err.name === 'InsecureContextError') {
    error.value += 'Camera access is only permitted in secure context. Use HTTPS or localhost rather than HTTP.'
  } else {
    error.value += err.message
  }
}

</script>

<style scoped>
.error {
  font-weight: bold;
  color: red;
}
</style>