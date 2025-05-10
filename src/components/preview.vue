<template>
  <div class="image-preview-container">
    <div class="image-preview-card">
      <div class="input-section">
        <label for="base64Input" class="input-label">Enter Base64 Encoded Image:</label>
        <textarea id="base64Input" v-model="inputData" class="base64-input"
          placeholder="Paste your base64 encoded image data here..." @input="handleInput"></textarea>
        <div class="input-actions">
          <button class="preview-button" @click="handleInput">Preview</button>
          <button class="clear-button" @click="clearInput">Clear</button>
        </div>
      </div>

      <div v-if="imageUrl" class="image-preview">
        <img :src="imageUrl" alt="Preview" class="preview-image" />
      </div>
      <div v-else class="image-placeholder">
        <div class="placeholder-icon">
          <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none"
            stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
            <circle cx="8.5" cy="8.5" r="1.5"></circle>
            <polyline points="21 15 16 10 5 21"></polyline>
          </svg>
        </div>
        <p>No image to preview</p>
      </div>

      <div class="image-info" v-if="imageUrl">
        <div class="info-item">
          <span class="info-label">Format:</span>
          <span class="info-value">{{ imageFormat || 'Unknown' }}</span>
        </div>
        <div class="info-item">
          <span class="info-label">Size:</span>
          <span class="info-value">{{ formatSize(imageSize) }}</span>
        </div>
        <div class="info-item" v-if="imageWidth && imageHeight">
          <span class="info-label">Dimensions:</span>
          <span class="info-value">{{ imageWidth }}×{{ imageHeight }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ImagePreview',
  props: {
    /**
     * Base64 encoded image data or data URL
     */
    imageData: {
      type: String,
      default: ''
    },
    /**
     * Image format (png, jpeg, etc.)
     */
    format: {
      type: String,
      default: ''
    },
    /**
     * Image size in bytes
     */
    size: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      imageUrl: '',
      imageFormat: '',
      imageSize: 0,
      inputData: '',
      imageWidth: 0,
      imageHeight: 0
    }
  },
  watch: {
    imageData: {
      immediate: true,
      handler(newValue) {
        if (newValue) {
          this.processImageData(newValue)
          this.inputData = newValue
        }
      }
    },
    format: {
      immediate: true,
      handler(newValue) {
        this.imageFormat = newValue || this.detectFormat(this.imageData)
      }
    },
    size: {
      immediate: true,
      handler(newValue) {
        this.imageSize = newValue
      }
    }
  },
  methods: {
    handleInput() {
      this.processImageData(this.inputData)

      // Calculate size if not provided as prop
      if (!this.size && this.inputData) {
        // Estimate size: base64 string length * 0.75 gives approximate byte size
        this.imageSize = Math.round(this.inputData.length * 0.75)
      }

      // Emit event with the new data
      this.$emit('input-change', {
        data: this.inputData,
        format: this.imageFormat,
        size: this.imageSize
      })
    },
    clearInput() {
      this.inputData = ''
      this.imageUrl = ''
      this.imageFormat = ''
      this.imageSize = 0
      this.$emit('input-clear')
    },
    processImageData(data) {
      if (!data) {
        this.imageUrl = ''
        return
      }

      // Clean up the input data (remove whitespace, newlines, etc.)
      const cleanData = data.trim().replace(/\s/g, '')

      // Check if the data is already a data URL
      if (cleanData.startsWith('data:image/')) {
        this.imageUrl = cleanData
      } else {
        // Assume it's base64 encoded data and try to determine format
        const format = this.detectFormat(cleanData) || 'png'
        this.imageUrl = `data:image/${format};base64,${cleanData}`
      }

      // Update format if not provided as prop
      if (!this.format) {
        this.imageFormat = this.detectFormat(cleanData) || 'unknown'
      }

      // Get image dimensions when loaded
      this.$nextTick(() => {
        const img = new Image()
        img.onload = () => {
          this.imageWidth = img.width
          this.imageHeight = img.height
        }
        img.src = this.imageUrl
      })
    },
    detectFormat(data) {
      // Simple format detection based on data signature
      if (!data) return ''

      // Check for PNG signature
      if (data.startsWith('iVBORw0KGgo') ||
        (data.startsWith('data:image/png'))) {
        return 'png'
      }

      // Check for JPEG signature
      if (data.startsWith('/9j/') ||
        data.startsWith('data:image/jpeg')) {
        return 'jpeg'
      }

      // Check for GIF signature
      if (data.startsWith('R0lGOD') ||
        data.startsWith('data:image/gif')) {
        return 'gif'
      }

      // Check for WebP signature
      if (data.startsWith('UklGR') ||
        data.startsWith('data:image/webp')) {
        return 'webp'
      }

      return ''
    },
    formatSize(bytes) {
      if (bytes === 0) return '0 Bytes'

      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))

      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    }
  }
}
</script>

<style scoped>
.image-preview-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  padding: 20px;
}

.image-preview-card {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  width: 100%;
  width: 1000px;
  transition: all 0.3s ease;
}

.image-preview-card:hover {
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.input-section {
  padding: 16px;
  border-bottom: 1px solid #f0f0f0;
}

.input-label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #495057;
}

.base64-input {
  width: 100%;
  min-height: 100px;
  padding: 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-family: monospace;
  font-size: 14px;
  resize: vertical;
  margin-bottom: 12px;
}

.base64-input:focus {
  outline: none;
  border-color: #80bdff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.input-actions {
  display: flex;
  gap: 10px;
}

.preview-button,
.clear-button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s ease;
}

.preview-button {
  background-color: #007bff;
  color: white;
}

.preview-button:hover {
  background-color: #0069d9;
}

.clear-button {
  background-color: #f8f9fa;
  color: #495057;
  border: 1px solid #ced4da;
}

.clear-button:hover {
  background-color: #e2e6ea;
}

.image-preview {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f8f9fa;
  min-width: 200px;
  max-width: 1000px;
  overflow: hidden;
}

.preview-image {
  max-width: 100%;
  object-fit: contain;
}

.image-placeholder {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: #f8f9fa;
  min-height: 200px;
  padding: 40px;
  color: #6c757d;
}

.placeholder-icon {
  margin-bottom: 16px;
  color: #adb5bd;
}

.image-info {
  padding: 16px;
  border-top: 1px solid #f0f0f0;
  background-color: #fafafa;
}

.info-item {
  display: flex;
  margin-bottom: 8px;
}

.info-item:last-child {
  margin-bottom: 0;
}

.info-label {
  font-weight: 600;
  color: #495057;
  width: 80px;
}

.info-value {
  color: #6c757d;
  margin-left: 5px;
}
</style>