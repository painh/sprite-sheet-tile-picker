<script setup lang="ts">
import { ref } from 'vue'
import ImageUploader from './components/ImageUploader.vue'
import GridSettings from './components/GridSettings.vue'
import ImageViewer from './components/ImageViewer.vue'
import CodePanel from './components/CodePanel.vue'

const imageUrl = ref<string | null>(null)
const gridX = ref(16)
const gridY = ref(16)
const gridMode = ref<'size' | 'count'>('size')
const showNumbers = ref(false)
const selectedTile = ref<{ index: number; row: number; col: number } | null>(null)
const imageName = ref('spritesheet')
const isDragging = ref(false)

const handleImageLoad = (url: string, name: string) => {
  imageUrl.value = url
  imageName.value = name.replace(/\.[^/.]+$/, '') || 'spritesheet'
  selectedTile.value = null
}

const handleTileClick = (tile: { index: number; row: number; col: number }) => {
  selectedTile.value = tile
}

let dragCounter = 0

const handleDragEnter = (e: DragEvent) => {
  e.preventDefault()
  dragCounter++
  isDragging.value = true
}

const handleDragOver = (e: DragEvent) => {
  e.preventDefault()
}

const handleDragLeave = (e: DragEvent) => {
  e.preventDefault()
  dragCounter--
  if (dragCounter === 0) {
    isDragging.value = false
  }
}

const handleDrop = (e: DragEvent) => {
  e.preventDefault()
  dragCounter = 0
  isDragging.value = false

  const files = e.dataTransfer?.files
  if (files && files.length > 0) {
    const file = files[0]
    if (file && file.type.startsWith('image/')) {
      const url = URL.createObjectURL(file)
      handleImageLoad(url, file.name)
    }
  }
}
</script>

<template>
  <div
    class="app-container"
    :class="{ 'dragging': isDragging }"
    @dragenter="handleDragEnter"
    @dragover="handleDragOver"
    @dragleave="handleDragLeave"
    @drop="handleDrop"
  >
    <header class="header">
      <h1>Sprite Sheet Tile Picker</h1>
      <div class="header-controls">
        <ImageUploader @image-loaded="handleImageLoad" />
        <GridSettings v-model:gridX="gridX" v-model:gridY="gridY" v-model:gridMode="gridMode" />
        <button
          class="toggle-numbers-btn"
          :class="{ active: showNumbers }"
          @click="showNumbers = !showNumbers"
        >
          {{ showNumbers ? 'Hide Numbers' : 'Show Numbers' }}
        </button>
      </div>
    </header>

    <main class="main-content">
      <div class="image-panel">
        <ImageViewer
          v-if="imageUrl"
          :imageUrl="imageUrl"
          :gridX="gridX"
          :gridY="gridY"
          :gridMode="gridMode"
          :showNumbers="showNumbers"
          :selectedTile="selectedTile"
          @tile-click="handleTileClick"
        />
        <div v-else class="placeholder">
          <div class="placeholder-content">
            <svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
              <rect x="3" y="3" width="18" height="18" rx="2" ry="2"/>
              <circle cx="8.5" cy="8.5" r="1.5"/>
              <polyline points="21 15 16 10 5 21"/>
            </svg>
            <p>Drag and drop an image here</p>
            <p class="sub">or use the upload button above</p>
          </div>
        </div>
      </div>

      <CodePanel
        :selectedTile="selectedTile"
        :gridX="gridX"
        :gridY="gridY"
        :gridMode="gridMode"
        :imageName="imageName"
      />
    </main>

    <div v-if="isDragging" class="drag-overlay">
      <div class="drag-content">
        <svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
          <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
          <polyline points="17 8 12 3 7 8"/>
          <line x1="12" y1="3" x2="12" y2="15"/>
        </svg>
        <p>Drop image to load</p>
      </div>
    </div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
  background-color: #1a1a2e;
  color: #eee;
  min-height: 100vh;
}

#app {
  min-height: 100vh;
}
</style>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  background: #16213e;
  padding: 1rem 1.5rem;
  border-bottom: 1px solid #0f3460;
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 1rem;
}

.header h1 {
  font-size: 1.25rem;
  color: #e94560;
  white-space: nowrap;
}

.header-controls {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.toggle-numbers-btn {
  padding: 0.5rem 1rem;
  background: #0f3460;
  border: 1px solid #e94560;
  color: #eee;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: all 0.2s;
}

.toggle-numbers-btn:hover {
  background: #e94560;
}

.toggle-numbers-btn.active {
  background: #e94560;
}

.main-content {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.image-panel {
  flex: 1;
  overflow: auto;
  padding: 1rem;
  background: #1a1a2e;
}

.placeholder {
  height: 100%;
  min-height: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px dashed #0f3460;
  border-radius: 8px;
  margin: 1rem;
}

.placeholder-content {
  text-align: center;
  color: #666;
}

.placeholder-content svg {
  margin-bottom: 1rem;
  opacity: 0.5;
}

.placeholder-content p {
  font-size: 1.125rem;
  margin-bottom: 0.5rem;
}

.placeholder-content .sub {
  font-size: 0.875rem;
  opacity: 0.7;
}

.drag-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(233, 69, 96, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.drag-content {
  text-align: center;
  color: white;
}

.drag-content svg {
  margin-bottom: 1rem;
}

.drag-content p {
  font-size: 1.5rem;
  font-weight: 600;
}

.dragging .placeholder {
  border-color: #e94560;
  background: rgba(233, 69, 96, 0.1);
}
</style>
