<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{
  'image-loaded': [url: string, name: string]
}>()

const fileInput = ref<HTMLInputElement | null>(null)

const handleClick = () => {
  fileInput.value?.click()
}

const handleFileChange = (e: Event) => {
  const target = e.target as HTMLInputElement
  const file = target.files?.[0]
  if (file && file.type.startsWith('image/')) {
    const url = URL.createObjectURL(file)
    emit('image-loaded', url, file.name)
  }
  target.value = ''
}
</script>

<template>
  <div class="image-uploader">
    <input
      ref="fileInput"
      type="file"
      accept="image/*"
      @change="handleFileChange"
      hidden
    />
    <button class="upload-btn" @click="handleClick">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
        <polyline points="17 8 12 3 7 8"/>
        <line x1="12" y1="3" x2="12" y2="15"/>
      </svg>
      Upload Image
    </button>
  </div>
</template>

<style scoped>
.upload-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  background: #e94560;
  border: none;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
  font-weight: 500;
  transition: background 0.2s;
}

.upload-btn:hover {
  background: #d63651;
}
</style>
