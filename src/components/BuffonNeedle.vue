<template>
  <div class="method-container">
    <div class="method-header">
      <h2 class="method-title">📍 蒲丰投针实验</h2>
      <p class="method-description">
        18世纪法国数学家蒲丰提出：在平行线间随机投掷针，针与线相交的概率为 2l/(πd)，
        其中 l 是针长，d 是平行线间距。由此可推导出 π = 2ln/(dc)，n 为总投掷次数，c 为相交次数。
      </p>
    </div>

    <div class="formula">
      π = (2 × 针长 × 投掷次数) / (线距 × 相交次数) = (2 × {{ needleLength }} × {{ totalNeedles }}) / ({{ lineSpacing }} × {{ crossingNeedles || 1 }})
    </div>

    <div class="canvas-container">
      <canvas
        ref="canvasRef"
        :width="width"
        :height="height"
        class="canvas"
      />
    </div>

    <div class="slider-container">
      <span class="slider-label">速度:</span>
      <input
        type="range"
        class="slider"
        min="1"
        max="20"
        v-model.number="speed"
      />
      <span class="result-value">{{ speed }} 针/帧</span>
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
          {{ crossingNeedles > 0 ? piEstimate.toFixed(10) : '等待数据...' }}
        </span>
      </div>
      <div class="result-row">
        <span class="result-label">真实 π 值</span>
        <span class="result-value">3.1415926536</span>
      </div>
      <div class="result-row">
        <span class="result-label">误差</span>
        <span class="result-value">
          {{ crossingNeedles > 0 ? `${error.toFixed(10)} (${errorPercent}%)` : '-' }}
        </span>
      </div>
      <div class="result-row">
        <span class="result-label">投掷总数</span>
        <span class="result-value">{{ totalNeedles.toLocaleString() }}</span>
      </div>
      <div class="result-row">
        <span class="result-label">相交次数</span>
        <span class="result-value" style="color: #7A8B7A">{{ crossingNeedles.toLocaleString() }}</span>
      </div>
      <div class="result-row">
        <span class="result-label">相交概率</span>
        <span class="result-value">{{ probability }}%</span>
      </div>
      <div class="result-row">
        <span class="result-label">理论概率</span>
        <span class="result-value">{{ theoreticalProbability }}%</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const canvasRef = ref(null)
const isRunning = ref(false)
const totalNeedles = ref(0)
const crossingNeedles = ref(0)
const speed = ref(5)
const needles = ref([])
let animationId = null

const width = 600
const height = 400
const lineSpacing = 60 // Distance between parallel lines (d)
const needleLength = 50 // Length of needle (l), must be <= lineSpacing

const piEstimate = computed(() => {
  if (crossingNeedles.value === 0) return 0
  return (2 * needleLength * totalNeedles.value) / (lineSpacing * crossingNeedles.value)
})

const error = computed(() => {
  if (crossingNeedles.value === 0) return 0
  return Math.abs(piEstimate.value - Math.PI)
})

const errorPercent = computed(() => {
  if (crossingNeedles.value === 0) return '0'
  return (error.value / Math.PI * 100).toFixed(4)
})

const probability = computed(() => {
  if (totalNeedles.value === 0) return '0'
  return (crossingNeedles.value / totalNeedles.value * 100).toFixed(2)
})

const theoreticalProbability = computed(() => {
  return ((2 * needleLength) / (Math.PI * lineSpacing) * 100).toFixed(2)
})

function drawBackground() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  ctx.fillStyle = '#FAFAF8'
  ctx.fillRect(0, 0, width, height)

  // Draw parallel lines
  ctx.strokeStyle = '#C4B8A8'
  ctx.lineWidth = 2
  for (let y = lineSpacing; y < height; y += lineSpacing) {
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(width, y)
    ctx.stroke()
  }

  // Draw all existing needles
  needles.value.forEach(needle => {
    ctx.beginPath()
    ctx.moveTo(needle.x1, needle.y1)
    ctx.lineTo(needle.x2, needle.y2)
    ctx.strokeStyle = needle.crosses ? '#7A8B7A' : '#A89888'
    ctx.lineWidth = 2
    ctx.stroke()
  })
}

function dropNeedle() {
  // Random center point
  const centerX = Math.random() * width
  const centerY = Math.random() * height

  // Random angle
  const angle = Math.random() * Math.PI

  // Calculate endpoints
  const halfLength = needleLength / 2
  const x1 = centerX - halfLength * Math.cos(angle)
  const y1 = centerY - halfLength * Math.sin(angle)
  const x2 = centerX + halfLength * Math.cos(angle)
  const y2 = centerY + halfLength * Math.sin(angle)

  // Check if needle crosses a line
  const minY = Math.min(y1, y2)
  const maxY = Math.max(y1, y2)

  let crosses = false
  for (let lineY = lineSpacing; lineY < height; lineY += lineSpacing) {
    if (minY <= lineY && maxY >= lineY) {
      crosses = true
      break
    }
  }

  return { x1, y1, x2, y2, crosses }
}

function addNeedle() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  const needle = dropNeedle()

  // Store needle
  needles.value.push(needle)

  // Draw needle
  ctx.beginPath()
  ctx.moveTo(needle.x1, needle.y1)
  ctx.lineTo(needle.x2, needle.y2)
  ctx.strokeStyle = needle.crosses ? '#7A8B7A' : '#A89888'
  ctx.lineWidth = 2
  ctx.stroke()

  // Update counts
  totalNeedles.value++
  if (needle.crosses) {
    crossingNeedles.value++
  }
}

function animate() {
  for (let i = 0; i < speed.value; i++) {
    addNeedle()
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
  totalNeedles.value = 0
  crossingNeedles.value = 0
  needles.value = []
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
