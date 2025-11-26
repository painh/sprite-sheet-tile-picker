<script setup lang="ts">
import { ref, computed } from 'vue'

const props = defineProps<{
  selectedTile: { index: number; row: number; col: number } | null
  gridX: number
  gridY: number
  gridMode: 'size' | 'count'
  imageName: string
}>()

const activeTab = ref<'phaser3' | 'phaser2'>('phaser3')
const copied = ref(false)

const frameInfo = computed(() => {
  if (props.gridMode === 'size') {
    return { width: props.gridX, height: props.gridY }
  }
  return { width: 'FRAME_WIDTH', height: 'FRAME_HEIGHT' }
})

const phaser3Code = computed(() => {
  if (!props.selectedTile) return ''

  const { index, row, col } = props.selectedTile
  const key = props.imageName
  const fw = frameInfo.value.width
  const fh = frameInfo.value.height

  return `// Phaser 3 - Load spritesheet in preload()
this.load.spritesheet('${key}', '${key}.png', {
  frameWidth: ${fw},
  frameHeight: ${fh}
});

// Create sprite with specific frame (index: ${index})
const sprite = this.add.sprite(x, y, '${key}', ${index});

// Or using setFrame on existing sprite
sprite.setFrame(${index});

// Tile position: row ${row}, col ${col}`
})

const phaser2Code = computed(() => {
  if (!props.selectedTile) return ''

  const { index, row, col } = props.selectedTile
  const key = props.imageName
  const fw = frameInfo.value.width
  const fh = frameInfo.value.height

  return `// Phaser 2 (CE) - Load spritesheet in preload()
game.load.spritesheet('${key}', '${key}.png', ${fw}, ${fh});

// Create sprite with specific frame (index: ${index})
const sprite = game.add.sprite(x, y, '${key}', ${index});

// Or using frame property on existing sprite
sprite.frame = ${index};

// Tile position: row ${row}, col ${col}`
})

const currentCode = computed(() => {
  return activeTab.value === 'phaser3' ? phaser3Code.value : phaser2Code.value
})

const copyToClipboard = async () => {
  if (!currentCode.value) return

  try {
    await navigator.clipboard.writeText(currentCode.value)
    copied.value = true
    setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch (err) {
    console.error('Failed to copy:', err)
  }
}
</script>

<template>
  <div class="code-panel">
    <div class="panel-header">
      <h2>Phaser Code</h2>
      <div class="tabs">
        <button
          :class="{ active: activeTab === 'phaser3' }"
          @click="activeTab = 'phaser3'"
        >
          Phaser 3
        </button>
        <button
          :class="{ active: activeTab === 'phaser2' }"
          @click="activeTab = 'phaser2'"
        >
          Phaser 2
        </button>
      </div>
    </div>

    <div class="panel-content">
      <div v-if="selectedTile" class="tile-info">
        <div class="info-row">
          <span class="label">Tile Index:</span>
          <span class="value highlight">{{ selectedTile.index }}</span>
        </div>
        <div class="info-row">
          <span class="label">Position:</span>
          <span class="value">Row {{ selectedTile.row }}, Col {{ selectedTile.col }}</span>
        </div>
      </div>

      <div v-if="selectedTile" class="code-container">
        <div class="code-header">
          <span>TypeScript</span>
          <button class="copy-btn" @click="copyToClipboard" :disabled="!currentCode">
            <svg v-if="!copied" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <rect x="9" y="9" width="13" height="13" rx="2" ry="2"/>
              <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
            </svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="20 6 9 17 4 12"/>
            </svg>
            {{ copied ? 'Copied!' : 'Copy' }}
          </button>
        </div>
        <pre class="code-block"><code>{{ currentCode }}</code></pre>
      </div>

      <div v-else class="empty-state">
        <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
          <polyline points="4 17 10 11 4 5"/>
          <line x1="12" y1="19" x2="20" y2="19"/>
        </svg>
        <p>Click on a tile to generate code</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.code-panel {
  width: 380px;
  min-width: 320px;
  background: #16213e;
  border-left: 1px solid #0f3460;
  display: flex;
  flex-direction: column;
}

.panel-header {
  padding: 1rem;
  border-bottom: 1px solid #0f3460;
}

.panel-header h2 {
  font-size: 1rem;
  margin-bottom: 0.75rem;
  color: #e94560;
}

.tabs {
  display: flex;
  gap: 0.5rem;
}

.tabs button {
  flex: 1;
  padding: 0.5rem;
  background: #0f3460;
  border: 1px solid transparent;
  color: #888;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.8rem;
  transition: all 0.2s;
}

.tabs button:hover {
  color: #eee;
}

.tabs button.active {
  background: #e94560;
  color: white;
  border-color: #e94560;
}

.panel-content {
  flex: 1;
  padding: 1rem;
  overflow: auto;
}

.tile-info {
  background: #0f3460;
  padding: 0.75rem;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.25rem 0;
}

.info-row .label {
  color: #888;
  font-size: 0.8rem;
}

.info-row .value {
  color: #eee;
  font-size: 0.875rem;
  font-family: monospace;
}

.info-row .value.highlight {
  color: #e94560;
  font-size: 1.25rem;
  font-weight: bold;
}

.code-container {
  background: #0a0a1a;
  border-radius: 4px;
  overflow: hidden;
}

.code-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0.75rem;
  background: #0f3460;
  font-size: 0.75rem;
  color: #888;
}

.copy-btn {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  padding: 0.25rem 0.5rem;
  background: transparent;
  border: 1px solid #e94560;
  color: #e94560;
  border-radius: 3px;
  cursor: pointer;
  font-size: 0.7rem;
  transition: all 0.2s;
}

.copy-btn:hover:not(:disabled) {
  background: #e94560;
  color: white;
}

.copy-btn:disabled {
  opacity: 0.5;
  cursor: default;
}

.code-block {
  padding: 1rem;
  margin: 0;
  overflow-x: auto;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 0.75rem;
  line-height: 1.6;
  color: #a8dadc;
  white-space: pre;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
  color: #444;
  text-align: center;
}

.empty-state svg {
  margin-bottom: 1rem;
  opacity: 0.5;
}

.empty-state p {
  font-size: 0.875rem;
}
</style>
