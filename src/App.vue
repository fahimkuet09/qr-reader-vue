<script setup>
import { QrcodeStream } from 'vue-qrcode-reader'

const onDetect = (detectedCodes) => {
  console.log(detectedCodes)
  alert('Student Id :'+ detectedCodes[0].rawValue)
}
</script>

<template>
  <QrcodeStream @detect="onDetect" />

</template>

<style scoped></style>
