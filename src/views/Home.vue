<template>
  <div>
    <p class="error">{{ error }}</p>

    <p class="text">
      Go to supermarket entrance to get the QR code
    </p>

    <p class="decode-result">
      Result: <b>{{ result }}</b>
    </p>

    <qrcode-capture @detect="onDetect" ></qrcode-capture>

    <p v-if="torchNotSupported" class="error">Torch not supported for active camera</p>
    
    <qrcode-stream @detect="onDetect" :track="paintOutline"  @camera-on="onCameraOn"  @error="onError" >
    <button :disabled="torchNotSupported">
      <img :src="'/flashlight.png'" alt="toggle torch" />
    </button>
    </qrcode-stream>

  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

import { QrcodeStream, QrcodeCapture } from 'vue-qrcode-reader'

// data
const result = ref('')
const error = ref('')
const torchNotSupported = ref(false)

// methods
const onCameraOn = (capabilities) => {
  console.log(capabilities);
  torchNotSupported.value = !capabilities.torch;
}

const onDetect = detectedCodes => {
  console.log(detectedCodes)

  const [ firstCode ] = detectedCodes
  result.value = firstCode.rawValue
  window.location.href = result.value;
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
button {
  position: absolute;
  left: 10px;
  top: 150px;
}

button img {
  width: 50px;
  height: 50px;
  margin-top: 5px;
}
.error {
  color: red;
  font-weight: bold;
}
</style>