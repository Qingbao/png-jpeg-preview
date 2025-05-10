<template>
  <div>
    <h1>Image Preview</h1>
    <ImagePreview :imageData="encodedImageData" :format="imageFormat" :size="imageSize" />
  </div>
</template>

<script>
import ImagePreview from './components/preview.vue'

export default {
  components: {
    ImagePreview
  },
  data() {
    return {
      encodedImageData: '', // Base64 encoded image data
      imageFormat: '',   // Optional: image format
      imageSize: 0      // Optional: image size in bytes
    }
  }
}
</script>