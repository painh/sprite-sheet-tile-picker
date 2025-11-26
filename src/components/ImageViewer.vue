<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

const props = defineProps<{
  imageUrl: string
  gridX: number
  gridY: number
  gridMode: 'size' | 'count'
  showNumbers: boolean
  selectedTile: { index: number; row: number; col: number } | null
}>()

const emit = defineEmits<{
  'tile-click': [tile: { index: number; row: number; col: number }]
}>()

const containerRef = ref<HTMLDivElement | null>(null)
const canvasRef = ref<HTMLCanvasElement | null>(null)
const image = ref<HTMLImageElement | null>(null)
const imageLoaded = ref(false)

// Viewport state
const scale = ref(1)
const offsetX = ref(0)
const offsetY = ref(0)
const isPanning = ref(false)
const lastMouseX = ref(0)
const lastMouseY = ref(0)
const hoveredTile = ref<number | null>(null)

const MIN_SCALE = 0.1
const MAX_SCALE = 10

const imageWidth = computed(() => image.value?.naturalWidth || 0)
const imageHeight = computed(() => image.value?.naturalHeight || 0)

// Grid mode: 'size' = gridX/Y is tile size in px, 'count' = gridX/Y is number of tiles
const cellWidth = computed(() => {
  if (props.gridMode === 'size') {
    return props.gridX
  }
  return imageWidth.value / props.gridX
})

const cellHeight = computed(() => {
  if (props.gridMode === 'size') {
    return props.gridY
  }
  return imageHeight.value / props.gridY
})

const colCount = computed(() => {
  if (props.gridMode === 'size') {
    return Math.floor(imageWidth.value / props.gridX) || 1
  }
  return props.gridX
})

const rowCount = computed(() => {
  if (props.gridMode === 'size') {
    return Math.floor(imageHeight.value / props.gridY) || 1
  }
  return props.gridY
})

const totalCells = computed(() => colCount.value * rowCount.value)

const loadImage = () => {
  const img = new Image()
  img.onload = () => {
    image.value = img
    imageLoaded.value = true
    resetView()
    draw()
  }
  img.src = props.imageUrl
}

const resetView = () => {
  if (!containerRef.value || !image.value) return

  const containerWidth = containerRef.value.clientWidth
  const containerHeight = containerRef.value.clientHeight
  const imgWidth = image.value.naturalWidth
  const imgHeight = image.value.naturalHeight

  // Fit image to container with some padding
  const scaleX = (containerWidth - 40) / imgWidth
  const scaleY = (containerHeight - 40) / imgHeight
  scale.value = Math.min(scaleX, scaleY, 1)

  // Center the image
  offsetX.value = (containerWidth - imgWidth * scale.value) / 2
  offsetY.value = (containerHeight - imgHeight * scale.value) / 2
}

const draw = () => {
  const canvas = canvasRef.value
  const ctx = canvas?.getContext('2d')
  if (!canvas || !ctx || !image.value || !containerRef.value) return

  // Set canvas size to container size
  const dpr = window.devicePixelRatio || 1
  const rect = containerRef.value.getBoundingClientRect()
  canvas.width = rect.width * dpr
  canvas.height = rect.height * dpr
  canvas.style.width = rect.width + 'px'
  canvas.style.height = rect.height + 'px'
  ctx.scale(dpr, dpr)

  // Clear canvas
  ctx.fillStyle = '#1a1a2e'
  ctx.fillRect(0, 0, rect.width, rect.height)

  // Draw checkerboard background for transparency
  ctx.save()
  ctx.translate(offsetX.value, offsetY.value)
  ctx.scale(scale.value, scale.value)

  const checkSize = 10
  for (let y = 0; y < imageHeight.value; y += checkSize) {
    for (let x = 0; x < imageWidth.value; x += checkSize) {
      ctx.fillStyle = ((x / checkSize + y / checkSize) % 2 === 0) ? '#2a2a3e' : '#1a1a2e'
      ctx.fillRect(x, y, checkSize, checkSize)
    }
  }

  // Draw image
  ctx.imageSmoothingEnabled = false
  ctx.drawImage(image.value, 0, 0)

  // Draw grid
  ctx.strokeStyle = 'rgba(233, 69, 96, 0.6)'
  ctx.lineWidth = 1 / scale.value

  // Vertical lines
  for (let i = 0; i <= colCount.value; i++) {
    const x = i * cellWidth.value
    ctx.beginPath()
    ctx.moveTo(x, 0)
    ctx.lineTo(x, imageHeight.value)
    ctx.stroke()
  }

  // Horizontal lines
  for (let j = 0; j <= rowCount.value; j++) {
    const y = j * cellHeight.value
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(imageWidth.value, y)
    ctx.stroke()
  }

  // Draw hovered tile
  if (hoveredTile.value !== null) {
    const col = hoveredTile.value % colCount.value
    const row = Math.floor(hoveredTile.value / colCount.value)
    ctx.fillStyle = 'rgba(233, 69, 96, 0.2)'
    ctx.fillRect(col * cellWidth.value, row * cellHeight.value, cellWidth.value, cellHeight.value)
  }

  // Draw selected tile
  if (props.selectedTile) {
    const { row, col } = props.selectedTile
    ctx.fillStyle = 'rgba(233, 69, 96, 0.4)'
    ctx.fillRect(col * cellWidth.value, row * cellHeight.value, cellWidth.value, cellHeight.value)
    ctx.strokeStyle = '#e94560'
    ctx.lineWidth = 2 / scale.value
    ctx.strokeRect(col * cellWidth.value, row * cellHeight.value, cellWidth.value, cellHeight.value)
  }

  // Draw numbers
  if (props.showNumbers) {
    const fontSize = Math.max(8, Math.min(14, Math.min(cellWidth.value, cellHeight.value) * 0.4))
    ctx.font = `bold ${fontSize}px monospace`
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'

    let index = 0
    for (let row = 0; row < rowCount.value; row++) {
      for (let col = 0; col < colCount.value; col++) {
        const x = col * cellWidth.value + cellWidth.value / 2
        const y = row * cellHeight.value + cellHeight.value / 2

        // Text shadow/stroke
        ctx.strokeStyle = 'rgba(0, 0, 0, 0.8)'
        ctx.lineWidth = 3 / scale.value
        ctx.strokeText(String(index), x, y)

        // Text fill
        ctx.fillStyle = props.selectedTile?.index === index ? '#fff' : 'rgba(255, 255, 255, 0.9)'
        ctx.fillText(String(index), x, y)

        index++
      }
    }
  }

  ctx.restore()
}

const screenToImage = (screenX: number, screenY: number) => {
  const rect = canvasRef.value?.getBoundingClientRect()
  if (!rect) return { x: 0, y: 0 }

  const x = (screenX - rect.left - offsetX.value) / scale.value
  const y = (screenY - rect.top - offsetY.value) / scale.value
  return { x, y }
}

const getTileAtPosition = (imgX: number, imgY: number) => {
  if (imgX < 0 || imgY < 0 || imgX >= imageWidth.value || imgY >= imageHeight.value) {
    return null
  }

  const col = Math.floor(imgX / cellWidth.value)
  const row = Math.floor(imgY / cellHeight.value)

  // Check if within valid grid bounds
  if (col >= colCount.value || row >= rowCount.value) {
    return null
  }

  const index = row * colCount.value + col

  return { index, row, col }
}

const handleWheel = (e: WheelEvent) => {
  e.preventDefault()

  const rect = canvasRef.value?.getBoundingClientRect()
  if (!rect) return

  const mouseX = e.clientX - rect.left
  const mouseY = e.clientY - rect.top

  // Calculate zoom
  const zoomFactor = e.deltaY > 0 ? 0.9 : 1.1
  const newScale = Math.max(MIN_SCALE, Math.min(MAX_SCALE, scale.value * zoomFactor))

  // Zoom towards mouse position
  const scaleRatio = newScale / scale.value
  offsetX.value = mouseX - (mouseX - offsetX.value) * scaleRatio
  offsetY.value = mouseY - (mouseY - offsetY.value) * scaleRatio

  scale.value = newScale
  draw()
}

const handleMouseDown = (e: MouseEvent) => {
  if (e.button === 1 || (e.button === 0 && e.shiftKey)) {
    // Middle click or Shift + Left click for panning
    isPanning.value = true
    lastMouseX.value = e.clientX
    lastMouseY.value = e.clientY
    e.preventDefault()
  }
}

const handleMouseMove = (e: MouseEvent) => {
  if (isPanning.value) {
    const dx = e.clientX - lastMouseX.value
    const dy = e.clientY - lastMouseY.value
    offsetX.value += dx
    offsetY.value += dy
    lastMouseX.value = e.clientX
    lastMouseY.value = e.clientY
    draw()
  } else {
    // Update hovered tile
    const imgPos = screenToImage(e.clientX, e.clientY)
    const tile = getTileAtPosition(imgPos.x, imgPos.y)
    const newHovered = tile?.index ?? null

    if (newHovered !== hoveredTile.value) {
      hoveredTile.value = newHovered
      draw()
    }
  }
}

const handleMouseUp = (e: MouseEvent) => {
  if (isPanning.value) {
    isPanning.value = false
    return
  }

  if (e.button === 0 && !e.shiftKey) {
    const imgPos = screenToImage(e.clientX, e.clientY)
    const tile = getTileAtPosition(imgPos.x, imgPos.y)
    if (tile) {
      emit('tile-click', tile)
    }
  }
}

const handleMouseLeave = () => {
  isPanning.value = false
  if (hoveredTile.value !== null) {
    hoveredTile.value = null
    draw()
  }
}

const zoomIn = () => {
  if (!containerRef.value) return
  const centerX = containerRef.value.clientWidth / 2
  const centerY = containerRef.value.clientHeight / 2

  const newScale = Math.min(MAX_SCALE, scale.value * 1.25)
  const scaleRatio = newScale / scale.value

  offsetX.value = centerX - (centerX - offsetX.value) * scaleRatio
  offsetY.value = centerY - (centerY - offsetY.value) * scaleRatio
  scale.value = newScale
  draw()
}

const zoomOut = () => {
  if (!containerRef.value) return
  const centerX = containerRef.value.clientWidth / 2
  const centerY = containerRef.value.clientHeight / 2

  const newScale = Math.max(MIN_SCALE, scale.value * 0.8)
  const scaleRatio = newScale / scale.value

  offsetX.value = centerX - (centerX - offsetX.value) * scaleRatio
  offsetY.value = centerY - (centerY - offsetY.value) * scaleRatio
  scale.value = newScale
  draw()
}

const handleResize = () => {
  draw()
}

watch(() => props.imageUrl, () => {
  imageLoaded.value = false
  loadImage()
})

watch([() => props.gridX, () => props.gridY, () => props.gridMode, () => props.showNumbers, () => props.selectedTile], () => {
  draw()
})

onMounted(() => {
  loadImage()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<template>
  <div class="image-viewer">
    <div class="toolbar">
      <button @click="zoomOut" title="Zoom Out">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"/>
          <line x1="21" y1="21" x2="16.65" y2="16.65"/>
          <line x1="8" y1="11" x2="14" y2="11"/>
        </svg>
      </button>
      <span class="zoom-level">{{ Math.round(scale * 100) }}%</span>
      <button @click="zoomIn" title="Zoom In">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"/>
          <line x1="21" y1="21" x2="16.65" y2="16.65"/>
          <line x1="11" y1="8" x2="11" y2="14"/>
          <line x1="8" y1="11" x2="14" y2="11"/>
        </svg>
      </button>
      <button @click="resetView" title="Reset View">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/>
          <polyline points="9 22 9 12 15 12 15 22"/>
        </svg>
      </button>
      <span class="separator">|</span>
      <span class="hint">Scroll to zoom, Shift+Drag to pan</span>
    </div>

    <div
      ref="containerRef"
      class="canvas-container"
      :class="{ panning: isPanning }"
    >
      <canvas
        ref="canvasRef"
        @wheel="handleWheel"
        @mousedown="handleMouseDown"
        @mousemove="handleMouseMove"
        @mouseup="handleMouseUp"
        @mouseleave="handleMouseLeave"
        @contextmenu.prevent
      />
    </div>

    <div class="image-info" v-if="imageLoaded">
      <span>Image: {{ imageWidth }} x {{ imageHeight }}px</span>
      <span>Cell: {{ Math.round(cellWidth) }} x {{ Math.round(cellHeight) }}px</span>
      <span>Total: {{ totalCells }} tiles</span>
    </div>
  </div>
</template>

<style scoped>
.image-viewer {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.toolbar {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem;
  background: #16213e;
  border-radius: 4px;
  margin-bottom: 0.5rem;
}

.toolbar button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: #0f3460;
  border: 1px solid #1a1a2e;
  color: #eee;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.toolbar button:hover {
  background: #e94560;
  border-color: #e94560;
}

.zoom-level {
  min-width: 50px;
  text-align: center;
  font-size: 0.8rem;
  color: #888;
  font-family: monospace;
}

.separator {
  color: #333;
  margin: 0 0.25rem;
}

.hint {
  font-size: 0.75rem;
  color: #666;
  margin-left: auto;
}

.canvas-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  border-radius: 4px;
  background: #1a1a2e;
  cursor: crosshair;
}

.canvas-container.panning {
  cursor: grabbing;
}

.canvas-container canvas {
  display: block;
  width: 100%;
  height: 100%;
}

.image-info {
  margin-top: 0.5rem;
  padding: 0.5rem 0.75rem;
  background: #16213e;
  border-radius: 4px;
  font-size: 0.75rem;
  color: #888;
  display: flex;
  gap: 1.5rem;
}
</style>
