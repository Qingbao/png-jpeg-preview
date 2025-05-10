<template>
  <div>
    <h1>Image Preview</h1>
    <ImagePreview :imageData="encodedImageData" :format="imageFormat" :size="imageSize" />
  </div>
</template>

<script setup lang="ts">
import ImagePreview from './components/preview.vue'

let encodedImageData: string; // Base64 encoded image data
let imageFormat: string;   // Optional: image format
let imageSize: number;  // Optional: image size in bytes

</script>