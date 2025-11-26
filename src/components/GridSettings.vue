<script setup lang="ts">
const gridX = defineModel<number>('gridX', { required: true })
const gridY = defineModel<number>('gridY', { required: true })
const gridMode = defineModel<'size' | 'count'>('gridMode', { required: true })

const handleInput = (e: Event, axis: 'x' | 'y') => {
  const target = e.target as HTMLInputElement
  const value = parseInt(target.value) || 1
  const maxVal = gridMode.value === 'size' ? 512 : 256
  const clampedValue = Math.max(1, Math.min(maxVal, value))

  if (axis === 'x') {
    gridX.value = clampedValue
  } else {
    gridY.value = clampedValue
  }
}

const toggleMode = () => {
  gridMode.value = gridMode.value === 'size' ? 'count' : 'size'
}
</script>

<template>
  <div class="grid-settings">
    <button class="mode-toggle" @click="toggleMode" :title="gridMode === 'size' ? 'Tile Size (px)' : 'Tile Count'">
      {{ gridMode === 'size' ? 'px' : '#' }}
    </button>
    <label>
      X:
      <input
        type="number"
        :value="gridX"
        @input="handleInput($event, 'x')"
        min="1"
        :max="gridMode === 'size' ? 512 : 256"
      />
    </label>
    <label>
      Y:
      <input
        type="number"
        :value="gridY"
        @input="handleInput($event, 'y')"
        min="1"
        :max="gridMode === 'size' ? 512 : 256"
      />
    </label>
    <span class="mode-label">{{ gridMode === 'size' ? 'Tile Size' : 'Tile Count' }}</span>
  </div>
</template>

<style scoped>
.grid-settings {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.mode-toggle {
  width: 32px;
  height: 32px;
  padding: 0;
  background: #e94560;
  border: none;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.75rem;
  font-weight: bold;
  transition: all 0.2s;
}

.mode-toggle:hover {
  background: #d63651;
}

label {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  font-size: 0.8rem;
  color: #aaa;
}

input {
  width: 55px;
  padding: 0.35rem 0.5rem;
  background: #0f3460;
  border: 1px solid #1a1a2e;
  color: #eee;
  border-radius: 4px;
  font-size: 0.8rem;
  text-align: center;
}

input:focus {
  outline: none;
  border-color: #e94560;
}

input::-webkit-inner-spin-button,
input::-webkit-outer-spin-button {
  opacity: 1;
}

.mode-label {
  font-size: 0.7rem;
  color: #666;
  margin-left: 0.25rem;
}
</style>
