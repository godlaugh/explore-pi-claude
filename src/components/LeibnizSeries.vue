<template>
  <div class="method-container">
    <div class="method-header">
      <h2 class="method-title">∑ 莱布尼茨级数</h2>
      <p class="method-description">
        使用无穷级数 π/4 = 1 - 1/3 + 1/5 - 1/7 + 1/9 - ... 来逼近圆周率。
        这个级数收敛较慢，但公式优美简洁。
      </p>
    </div>

    <div class="formula">
      {{ seriesDisplay }}
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
        max="100"
        v-model.number="speed"
      />
      <span class="result-value">{{ speed }} 项/帧</span>
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
        <span class="result-label">计算项数</span>
        <span class="result-value">{{ terms.toLocaleString() }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'

const canvasRef = ref(null)
const isRunning = ref(false)
const terms = ref(0)
const speed = ref(5)
const seriesTerms = ref([])
const sum = ref(0)
let animationId = null

const width = 600
const height = 300

const piEstimate = computed(() => sum.value * 4)

const error = computed(() => {
  if (terms.value === 0) return 0
  return Math.abs(piEstimate.value - Math.PI)
})

const errorPercent = computed(() => {
  if (terms.value === 0) return '0'
  return (error.value / Math.PI * 100).toFixed(6)
})

const seriesDisplay = computed(() => {
  if (terms.value === 0) return 'π/4 = 1 − 1/3 + 1/5 − 1/7 + 1/9 − ...'
  const n = Math.min(terms.value, 6)
  let display = 'π/4 = '
  for (let i = 0; i < n; i++) {
    if (i > 0) display += i % 2 === 0 ? ' + ' : ' − '
    display += `1/${2 * i + 1}`
  }
  if (terms.value > 6) display += ' ...'
  return display
})

function drawChart(data, currentPi) {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  ctx.fillStyle = '#FAFAF8'
  ctx.fillRect(0, 0, width, height)

  const padding = 50
  const chartWidth = width - padding * 2
  const chartHeight = height - padding * 2

  // Draw axes
  ctx.strokeStyle = '#C4B8A8'
  ctx.lineWidth = 1
  ctx.beginPath()
  ctx.moveTo(padding, padding)
  ctx.lineTo(padding, height - padding)
  ctx.lineTo(width - padding, height - padding)
  ctx.stroke()

  // Draw π reference line
  const piY = padding + chartHeight * (1 - (Math.PI - 2.5) / 1.5)
  ctx.strokeStyle = '#5B7A8C'
  ctx.lineWidth = 2
  ctx.setLineDash([5, 5])
  ctx.beginPath()
  ctx.moveTo(padding, piY)
  ctx.lineTo(width - padding, piY)
  ctx.stroke()
  ctx.setLineDash([])

  // Label
  ctx.fillStyle = '#5B7A8C'
  ctx.font = '14px Noto Serif SC'
  ctx.textAlign = 'right'
  ctx.fillText('π = 3.14159...', width - padding - 10, piY - 8)

  // Draw data line
  if (data.length > 1) {
    ctx.strokeStyle = '#7A8B7A'
    ctx.lineWidth = 2
    ctx.beginPath()

    const step = chartWidth / Math.max(data.length - 1, 1)

    data.forEach((value, i) => {
      const x = padding + i * step
      const normalizedValue = (value - 2.5) / 1.5
      const y = padding + chartHeight * (1 - Math.max(0, Math.min(1, normalizedValue)))

      if (i === 0) {
        ctx.moveTo(x, y)
      } else {
        ctx.lineTo(x, y)
      }
    })
    ctx.stroke()

    // Draw current point
    if (data.length > 0) {
      const lastX = padding + (data.length - 1) * step
      const lastValue = data[data.length - 1]
      const normalizedValue = (lastValue - 2.5) / 1.5
      const lastY = padding + chartHeight * (1 - Math.max(0, Math.min(1, normalizedValue)))

      ctx.beginPath()
      ctx.arc(lastX, lastY, 5, 0, Math.PI * 2)
      ctx.fillStyle = '#5D4E6D'
      ctx.fill()
    }
  }

  // Y-axis labels
  ctx.fillStyle = '#6B6B6B'
  ctx.font = '12px Courier New'
  ctx.textAlign = 'right'
  for (let i = 0; i <= 3; i++) {
    const value = 2.5 + i * 0.5
    const y = padding + chartHeight * (1 - i / 3)
    ctx.fillText(value.toFixed(1), padding - 10, y + 4)
  }

  // X-axis label
  ctx.textAlign = 'center'
  ctx.fillText(`项数: ${data.length}`, width / 2, height - 15)
}

function animate() {
  for (let i = 0; i < speed.value; i++) {
    const n = terms.value
    const term = (n % 2 === 0 ? 1 : -1) / (2 * n + 1)
    sum.value += term
    terms.value++
  }

  const newPi = sum.value * 4
  seriesTerms.value = [...seriesTerms.value.slice(-199), newPi]
  drawChart(seriesTerms.value, newPi)

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
  terms.value = 0
  sum.value = 0
  seriesTerms.value = []
  drawChart([], 0)
}

onMounted(() => {
  drawChart([], 0)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
})
</script>
