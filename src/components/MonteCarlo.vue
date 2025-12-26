<template>
  <div class="method-container">
    <div class="method-header">
      <h2 class="method-title">🎲 蒙特卡洛方法</h2>
      <p class="method-description">
        在正方形内随机撒点，统计落在内切圆中的点数。
        由于圆面积与正方形面积之比为 π/4，因此 π ≈ 4 × (圆内点数 / 总点数)
      </p>
    </div>

    <div class="formula">
      π ≈ 4 × (圆内点数 / 总点数) = 4 × ({{ pointsInside }} / {{ totalPoints || 1 }})
    </div>

    <div class="canvas-container">
      <canvas
        ref="canvasRef"
        :width="size"
        :height="size"
        class="canvas"
      />
    </div>

    <div class="slider-container">
      <span class="slider-label">速度:</span>
      <input
        type="range"
        class="slider"
        min="1"
        max="50"
        v-model.number="speed"
      />
      <span class="result-value">{{ speed }} 点/帧</span>
    </div>

    <div class="controls">
      <button v-if="!isRunning" class="btn btn-primary" @click="start">
        ▶ 开始
      </button>
      <button v-else class="btn btn-primary" @click="pause">
        ⏸ 暂停
      </button>
      <button class="btn btn-secondary" @click="reset">
        ↺ 重置
      </button>
    </div>

    <div class="result-panel">
      <div class="result-row">
        <span class="result-label">估算的 π 值</span>
        <span class="result-value result-highlight">
          {{ piEstimate.toFixed(10) }}
        </span>
      </div>
      <div class="result-row">
        <span class="result-label">真实 π 值</span>
        <span class="result-value">3.1415926536</span>
      </div>
      <div class="result-row">
        <span class="result-label">误差</span>
        <span class="result-value">{{ error.toFixed(10) }} ({{ errorPercent }}%)</span>
      </div>
      <div class="result-row">
        <span class="result-label">总点数</span>
        <span class="result-value">{{ totalPoints.toLocaleString() }}</span>
      </div>
      <div class="result-row">
        <span class="result-label">圆内点数</span>
        <span class="result-value" style="color: #7A8B7A">{{ pointsInside.toLocaleString() }}</span>
      </div>
      <div class="result-row">
        <span class="result-label">圆外点数</span>
        <span class="result-value" style="color: #A89888">{{ (totalPoints - pointsInside).toLocaleString() }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'

const canvasRef = ref(null)
const isRunning = ref(false)
const pointsInside = ref(0)
const totalPoints = ref(0)
const speed = ref(10)
const points = ref([])
let animationId = null

const size = 400
const radius = size / 2 - 20
const centerX = size / 2
const centerY = size / 2

const piEstimate = computed(() => {
  if (totalPoints.value === 0) return 0
  return (pointsInside.value / totalPoints.value) * 4
})

const error = computed(() => {
  if (totalPoints.value === 0) return 0
  return Math.abs(piEstimate.value - Math.PI)
})

const errorPercent = computed(() => {
  if (totalPoints.value === 0) return '0'
  return (error.value / Math.PI * 100).toFixed(4)
})

function drawBackground() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  // Clear canvas
  ctx.fillStyle = '#FAFAF8'
  ctx.fillRect(0, 0, size, size)

  // Draw square border
  ctx.strokeStyle = '#C4B8A8'
  ctx.lineWidth = 2
  ctx.strokeRect(centerX - radius, centerY - radius, radius * 2, radius * 2)

  // Draw circle
  ctx.beginPath()
  ctx.arc(centerX, centerY, radius, 0, Math.PI * 2)
  ctx.strokeStyle = '#5B7A8C'
  ctx.lineWidth = 2
  ctx.stroke()

  // Draw all existing points
  points.value.forEach(point => {
    ctx.beginPath()
    ctx.arc(point.x, point.y, 2, 0, Math.PI * 2)
    ctx.fillStyle = point.inside ? '#7A8B7A' : '#A89888'
    ctx.fill()
  })
}

function addPoint() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  // Generate random point within the square
  const x = centerX - radius + Math.random() * radius * 2
  const y = centerY - radius + Math.random() * radius * 2

  // Check if inside circle
  const distanceSquared = (x - centerX) ** 2 + (y - centerY) ** 2
  const inside = distanceSquared <= radius ** 2

  // Store point
  points.value.push({ x, y, inside })

  // Draw point
  ctx.beginPath()
  ctx.arc(x, y, 2, 0, Math.PI * 2)
  ctx.fillStyle = inside ? '#7A8B7A' : '#A89888'
  ctx.fill()

  // Update counts
  totalPoints.value++
  if (inside) {
    pointsInside.value++
  }
}

function animate() {
  for (let i = 0; i < speed.value; i++) {
    addPoint()
  }
  animationId = requestAnimationFrame(animate)
}

function start() {
  isRunning.value = true
  animate()
}

function pause() {
  isRunning.value = false
  if (animationId) {
    cancelAnimationFrame(animationId)
    animationId = null
  }
}

function reset() {
  pause()
  pointsInside.value = 0
  totalPoints.value = 0
  points.value = []
  drawBackground()
}

onMounted(() => {
  drawBackground()
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
})
</script>
