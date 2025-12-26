<template>
  <div class="app">
    <header class="header">
      <h1 class="title">
        <span class="pi-symbol">π</span>
        圆周率求解演示
      </h1>
      <p class="subtitle">探索数学之美，见证无理数的奥秘</p>
    </header>

    <nav class="nav">
      <button
        v-for="method in methods"
        :key="method.id"
        :class="['nav-btn', { active: activeMethod === method.id }]"
        @click="activeMethod = method.id"
      >
        <span class="nav-icon">{{ method.icon }}</span>
        <span class="nav-name">{{ method.name }}</span>
        <span class="nav-desc">{{ method.description }}</span>
      </button>
    </nav>

    <main class="main">
      <MonteCarlo v-if="activeMethod === 'monte-carlo'" />
      <LeibnizSeries v-else-if="activeMethod === 'leibniz'" />
      <ArchimedesMethod v-else-if="activeMethod === 'archimedes'" />
      <BuffonNeedle v-else-if="activeMethod === 'buffon'" />
    </main>

    <footer class="footer">
      <p>π ≈ 3.14159265358979323846264338327950288...</p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import MonteCarlo from './components/MonteCarlo.vue'
import LeibnizSeries from './components/LeibnizSeries.vue'
import ArchimedesMethod from './components/ArchimedesMethod.vue'
import BuffonNeedle from './components/BuffonNeedle.vue'

const activeMethod = ref('monte-carlo')

const methods = [
  { id: 'monte-carlo', name: '蒙特卡洛法', icon: '🎲', description: '随机投点，以概率估算π' },
  { id: 'leibniz', name: '莱布尼茨级数', icon: '∑', description: '无穷级数逼近π/4' },
  { id: 'archimedes', name: '阿基米德割圆', icon: '⬡', description: '正多边形逼近圆周' },
  { id: 'buffon', name: '蒲丰投针', icon: '📍', description: '投针实验求π' },
]
</script>
