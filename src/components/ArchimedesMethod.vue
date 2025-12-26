<template>
  <div class="method-container">
    <div class="method-header">
      <h2 class="method-title">⬡ 阿基米德割圆术</h2>
      <p class="method-description">
        用正多边形的周长来逼近圆的周长。内切多边形周长 &lt; 圆周长 &lt; 外切多边形周长。
        当边数趋于无穷时，多边形周长趋近于圆周长，从而得到 π 的值。
      </p>
    </div>

    <div class="formula">
      内切周长/(2r) &lt; π &lt; 外切周长/(2r)
      <br />
      {{ piLower.toFixed(8) }} &lt; π &lt; {{ piUpper.toFixed(8) }}
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
      <span class="slider-label">边数:</span>
      <input
        type="range"
        class="slider"
        min="3"
        max="200"
        :value="Math.min(sides, 200)"
        @input="sides = Number($event.target.value)"
      />
      <span class="result-value">{{ sides }}</span>
    </div>

    <div class="controls">
      <button class="btn btn-secondary" @click="decrease" :disabled="sides <= 3">
        边数 ÷ 2
      </button>
      <button class="btn btn-primary" @click="toggleAnimate">
        {{ isAnimating ? '⏸ 停止' : '▶ 自动演示' }}
      </button>
      <button class="btn btn-secondary" @click="increase" :disabled="sides >= 1000">
        边数 × 2
      </button>
      <button class="btn btn-secondary" @click="reset">
        ↺ 重置
      </button>
    </div>

    <div class="result-panel">
      <div class="result-row">
        <span class="result-label">估算的 π 值 (平均)</span>
        <span class="result-value result-highlight">
          {{ piEstimate.toFixed(10) }}
        </span>
      </div>
      <div class="result-row">
        <span class="result-label">π 下界 (内切)</span>
        <span class="result-value" style="color: #7A8B7A">
          {{ piLower.toFixed(10) }}
        </span>
      </div>
      <div class="result-row">
        <span class="result-label">π 上界 (外切)</span>
        <span class="result-value" style="color: #A89888">
          {{ piUpper.toFixed(10) }}
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
        <span class="result-label">边数</span>
        <span class="result-value">{{ sides.toLocaleString() }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'

const canvasRef = ref(null)
const sides = ref(6)
const isAnimating = ref(false)
let animationTimeout = null

const size = 400
const centerX = size / 2
const centerY = size / 2
const radius = 150

const perimeters = computed(() => {
  const n = sides.value
  // Inner polygon (inscribed)
  const innerSideLength = 2 * radius * Math.sin(Math.PI / n)
  const inner = n * innerSideLength

  // Outer polygon (circumscribed)
  const outerSideLength = 2 * radius * Math.tan(Math.PI / n)
  const outer = n * outerSideLength

  return { inner, outer }
})

const piLower = computed(() => perimeters.value.inner / (2 * radius))
const piUpper = computed(() => perimeters.value.outer / (2 * radius))
const piEstimate = computed(() => (piLower.value + piUpper.value) / 2)

const error = computed(() => Math.abs(piEstimate.value - Math.PI))
const errorPercent = computed(() => (error.value / Math.PI * 100).toFixed(8))

function drawVisualization(n) {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  ctx.fillStyle = '#FAFAF8'
  ctx.fillRect(0, 0, size, size)

  // Draw outer polygon (circumscribed) - filled
  ctx.beginPath()
  for (let i = 0; i < n; i++) {
    const angle = (2 * Math.PI * i) / n - Math.PI / 2
    const outerRadius = radius / Math.cos(Math.PI / n)
    const x = centerX + outerRadius * Math.cos(angle)
    const y = centerY + outerRadius * Math.sin(angle)
    if (i === 0) ctx.moveTo(x, y)
    else ctx.lineTo(x, y)
  }
  ctx.closePath()
  ctx.fillStyle = '#E8E4DF'
  ctx.fill()
  ctx.strokeStyle = '#A89888'
  ctx.lineWidth = 2
  ctx.stroke()

  // Draw circle
  ctx.beginPath()
  ctx.arc(centerX, centerY, radius, 0, Math.PI * 2)
  ctx.fillStyle = '#D8E4D8'
  ctx.fill()
  ctx.strokeStyle = '#5B7A8C'
  ctx.lineWidth = 3
  ctx.stroke()

  // Draw inner polygon (inscribed) - filled
  ctx.beginPath()
  for (let i = 0; i < n; i++) {
    const angle = (2 * Math.PI * i) / n - Math.PI / 2
    const x = centerX + radius * Math.cos(angle)
    const y = centerY + radius * Math.sin(angle)
    if (i === 0) ctx.moveTo(x, y)
    else ctx.lineTo(x, y)
  }
  ctx.closePath()
  ctx.fillStyle = '#C8D8C8'
  ctx.fill()
  ctx.strokeStyle = '#7A8B7A'
  ctx.lineWidth = 2
  ctx.stroke()

  // Draw center point
  ctx.beginPath()
  ctx.arc(centerX, centerY, 4, 0, Math.PI * 2)
  ctx.fillStyle = '#5D4E6D'
  ctx.fill()

  // Draw radius line
  ctx.beginPath()
  ctx.moveTo(centerX, centerY)
  ctx.lineTo(centerX + radius, centerY)
  ctx.strokeStyle = '#5D4E6D'
  ctx.lineWidth = 2
  ctx.setLineDash([4, 4])
  ctx.stroke()
  ctx.setLineDash([])

  // Draw labels
  ctx.fillStyle = '#4A4A4A'
  ctx.font = '14px Noto Serif SC'
  ctx.textAlign = 'center'
  ctx.fillText(`正 ${n} 边形`, centerX, size - 20)
}

watch(sides, (n) => {
  drawVisualization(n)
})

function animate() {
  if (sides.value >= 1000) {
    isAnimating.value = false
    return
  }

  sides.value = Math.min(sides.value + Math.ceil(sides.value * 0.1), 1000)

  animationTimeout = setTimeout(() => {
    requestAnimationFrame(animate)
  }, 100)
}

function toggleAnimate() {
  if (isAnimating.value) {
    isAnimating.value = false
    if (animationTimeout) {
      clearTimeout(animationTimeout)
    }
  } else {
    isAnimating.value = true
    sides.value = 6
    animate()
  }
}

function increase() {
  sides.value = Math.min(sides.value * 2, 1000)
}

function decrease() {
  sides.value = Math.max(Math.floor(sides.value / 2), 3)
}

function reset() {
  isAnimating.value = false
  if (animationTimeout) {
    clearTimeout(animationTimeout)
  }
  sides.value = 6
}

onMounted(() => {
  drawVisualization(sides.value)
})

onUnmounted(() => {
  if (animationTimeout) {
    clearTimeout(animationTimeout)
  }
})
</script>
