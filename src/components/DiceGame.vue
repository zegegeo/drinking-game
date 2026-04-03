<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

defineEmits(['goBack'])

// 骰子配置
const diceCount = ref(2)
const diceValues = ref([])
const isRolling = ref(false)
const rollHistory = ref([])
const showStats = ref(false)
const showResult = ref(false)

// 每颗骰子独立的3D旋转状态
const diceRotations = ref([])

// 骰子点数位置配置
const diceDotPositions = {
  1: [[50, 50]],
  2: [[25, 25], [75, 75]],
  3: [[25, 25], [50, 50], [75, 75]],
  4: [[25, 25], [75, 25], [25, 75], [75, 75]],
  5: [[25, 25], [75, 25], [50, 50], [25, 75], [75, 75]],
  6: [[25, 25], [75, 25], [25, 50], [75, 50], [25, 75], [75, 75]]
}

const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(30)
  }
}

const totalPoints = computed(() => {
  return diceValues.value.reduce((sum, val) => sum + val, 0)
})

const diceStats = computed(() => {
  const stats = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0, 6: 0 }
  rollHistory.value.forEach(roll => {
    roll.values.forEach(v => {
      stats[v] = (stats[v] || 0) + 1
    })
  })
  return stats
})

const totalRolls = computed(() => {
  return rollHistory.value.reduce((sum, roll) => sum + roll.values.length, 0)
})

const mostCommon = computed(() => {
  const stats = diceStats.value
  let max = 0
  let result = '-'
  for (let i = 1; i <= 6; i++) {
    if (stats[i] > max) {
      max = stats[i]
      result = i
    }
  }
  return result
})

const averageTotal = computed(() => {
  if (rollHistory.value.length === 0) return '-'
  const sum = rollHistory.value.reduce((s, r) => s + r.total, 0)
  return (sum / rollHistory.value.length).toFixed(1)
})

// 生成随机3D旋转角度（用于动画中间帧）
const randomRotation = () => ({
  x: Math.random() * 720 - 360,
  y: Math.random() * 720 - 360,
  z: Math.random() * 360 - 180
})

const rollDice = () => {
  if (isRolling.value) return
  triggerHaptic()
  isRolling.value = true
  showResult.value = false

  // 初始化骰子值和旋转
  diceValues.value = new Array(diceCount.value).fill(1)
  diceRotations.value = new Array(diceCount.value).fill(null).map(() => ({
    x: 0, y: 0, z: 0, phase: 'idle'
  }))

  // 每颗骰子独立滚动
  const rollPromises = diceValues.value.map((_, index) => {
    return new Promise(resolve => {
      let rolls = 0
      const maxRolls = 12 + Math.floor(Math.random() * 6)
      const baseInterval = 60

      // 启动旋转动画阶段
      diceRotations.value[index].phase = 'spinning'

      const interval = setInterval(() => {
        diceValues.value[index] = Math.floor(Math.random() * 6) + 1
        const rot = randomRotation()
        diceRotations.value[index] = {
          x: rot.x,
          y: rot.y,
          z: rot.z,
          phase: 'spinning'
        }
        rolls++

        if (rolls >= maxRolls) {
          clearInterval(interval)
          // 落地阶段
          diceRotations.value[index] = {
            x: 0, y: 0, z: 0, phase: 'landing'
          }
          triggerHaptic()
          resolve()
        }
      }, baseInterval + index * 20 + (rolls > 8 ? (rolls - 8) * 15 : 0))
    })
  })

  Promise.all(rollPromises).then(() => {
    isRolling.value = false
    showResult.value = true
    rollHistory.value.unshift({
      values: [...diceValues.value],
      total: totalPoints.value,
      time: Date.now()
    })
    if (rollHistory.value.length > 50) rollHistory.value.pop()

    // 落地后短暂延迟，重置phase
    setTimeout(() => {
      diceRotations.value = diceRotations.value.map(() => ({
        x: 0, y: 0, z: 0, phase: 'idle'
      }))
    }, 400)
  })
}

const reset = () => {
  triggerHaptic()
  diceValues.value = []
  diceRotations.value = []
  rollHistory.value = []
  showResult.value = false
  showStats.value = false
}

const clearHistory = () => {
  triggerHaptic()
  rollHistory.value = []
}

// 统计面板触摸拖拽关闭
const sheetStartY = ref(0)
const sheetTranslateY = ref(0)
const isDragging = ref(false)

const onSheetTouchStart = (e) => {
  sheetStartY.value = e.touches[0].clientY
  isDragging.value = true
  sheetTranslateY.value = 0
}

const onSheetTouchMove = (e) => {
  if (!isDragging.value) return
  const dy = e.touches[0].clientY - sheetStartY.value
  if (dy > 0) {
    sheetTranslateY.value = dy
  }
}

const onSheetTouchEnd = () => {
  isDragging.value = false
  if (sheetTranslateY.value > 120) {
    showStats.value = false
  }
  sheetTranslateY.value = 0
}

const formatTime = (timestamp) => {
  const d = new Date(timestamp)
  return `${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}:${String(d.getSeconds()).padStart(2, '0')}`
}

onMounted(() => {
  document.body.style.overflow = 'hidden'
})

onUnmounted(() => {
  document.body.style.overflow = ''
})
</script>

<template>
  <div class="dice-game">
    <!-- 顶部标题栏 -->
    <header class="top-bar">
      <button class="icon-btn" @click="$emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="title">掷骰子</h1>
      <button class="icon-btn" @click="showStats = true">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M3 3v18h18"/>
          <path d="M18 17V9"/>
          <path d="M13 17V5"/>
          <path d="M8 17v-3"/>
        </svg>
      </button>
    </header>

    <!-- 主游戏区域 -->
    <main class="main-area">
      <!-- 骰子数量选择 -->
      <section class="selector-section">
        <div class="selector-label">选择骰子数量</div>
        <div class="selector-pills">
          <button
            v-for="n in 6"
            :key="n"
            class="pill pill-stagger"
            :class="{ active: diceCount === n }"
            :style="{ animationDelay: `${(n - 1) * 0.05}s` }"
            @click="diceCount = n; triggerHaptic()"
          >
            {{ n }}
          </button>
        </div>
      </section>

      <!-- 骰子展示区 -->
      <section class="dice-stage">
        <div class="dice-container" v-if="diceValues.length > 0">
          <div
            v-for="(value, index) in diceValues"
            :key="index"
            class="dice-wrapper"
            :style="{ animationDelay: `${index * 0.08}s` }"
          >
            <div
              class="dice-3d"
              :class="{
                spinning: diceRotations[index]?.phase === 'spinning',
                landing: diceRotations[index]?.phase === 'landing'
              }"
              :style="diceRotations[index]?.phase === 'spinning' ? {
                transform: `rotateX(${diceRotations[index].x}deg) rotateY(${diceRotations[index].y}deg) rotateZ(${diceRotations[index].z}deg)`
              } : {}"
            >
              <div class="dice-face-front">
                <div
                  v-for="(dot, di) in diceDotPositions[value] || []"
                  :key="di"
                  class="dot"
                  :style="{ top: `${dot[0]}%`, left: `${dot[1]}%` }"
                ></div>
              </div>
            </div>
            <!-- 骰子阴影 -->
            <div
              class="dice-shadow"
              :class="{ active: diceRotations[index]?.phase !== 'spinning' }"
            ></div>
          </div>
        </div>

        <!-- 空状态 -->
        <div v-else class="empty-placeholder">
          <div class="empty-dice-icon">
            <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="rgba(255,255,255,0.2)" stroke-width="1.5">
              <rect x="3" y="3" width="18" height="18" rx="3"/>
              <circle cx="8.5" cy="8.5" r="1.2"/>
              <circle cx="15.5" cy="8.5" r="1.2"/>
              <circle cx="12" cy="12" r="1.2"/>
              <circle cx="8.5" cy="15.5" r="1.2"/>
              <circle cx="15.5" cy="15.5" r="1.2"/>
            </svg>
          </div>
          <span class="empty-hint">点击下方按钮开始掷骰子</span>
        </div>
      </section>

      <!-- 结果卡片 -->
      <transition name="result-pop">
        <section v-if="showResult && !isRolling" class="result-card">
          <div class="result-top">
            <span class="result-label">总点数</span>
            <span class="result-total">{{ totalPoints }}</span>
          </div>
          <div class="result-detail">
            <span
              v-for="(value, index) in diceValues"
              :key="index"
              class="result-chip"
            >{{ value }}</span>
          </div>
        </section>
      </transition>

      <!-- 历史记录 -->
      <section v-if="rollHistory.length > 0" class="history-section">
        <div class="section-header">
          <span class="section-title">历史记录</span>
          <span class="section-count">{{ rollHistory.length }} 次</span>
        </div>
        <div class="history-list">
          <div
            v-for="(roll, index) in rollHistory"
            :key="roll.time"
            class="history-row history-stagger"
            :class="{ latest: index === 0 }"
            :style="{ animationDelay: `${index * 0.04}s` }"
          >
            <div class="history-left">
              <span class="history-index">#{{ rollHistory.length - index }}</span>
              <div class="history-dice-group">
                <span
                  v-for="(v, vi) in roll.values"
                  :key="vi"
                  class="history-dice-val"
                >{{ v }}</span>
              </div>
            </div>
            <div class="history-right">
              <span class="history-total">{{ roll.total }}</span>
              <span class="history-time">{{ formatTime(roll.time) }}</span>
            </div>
          </div>
        </div>
      </section>
    </main>

    <!-- 底部操作栏 -->
    <footer class="action-bar">
      <button class="btn-reset" @click="reset" :disabled="isRolling || diceValues.length === 0">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 12"/>
          <path d="M3 3v9h9"/>
        </svg>
      </button>
      <button class="btn-roll" @click="rollDice" :disabled="isRolling">
        <span class="btn-shimmer"></span>
        <span class="btn-roll-inner">
          <svg
            v-if="isRolling"
            class="roll-spinner"
            width="22" height="22" viewBox="0 0 24 24"
            fill="none" stroke="currentColor" stroke-width="2.5"
          >
            <path d="M12 2v4M12 18v4M4.93 4.93l2.83 2.83M16.24 16.24l2.83 2.83M2 12h4M18 12h4M4.93 19.07l2.83-2.83M16.24 7.76l2.83-2.83"/>
          </svg>
          <svg
            v-else
            width="22" height="22" viewBox="0 0 24 24"
            fill="none" stroke="currentColor" stroke-width="2.5"
          >
            <rect x="3" y="3" width="18" height="18" rx="3"/>
            <circle cx="8.5" cy="8.5" r="1.2"/>
            <circle cx="15.5" cy="8.5" r="1.2"/>
            <circle cx="12" cy="12" r="1.2"/>
            <circle cx="8.5" cy="15.5" r="1.2"/>
            <circle cx="15.5" cy="15.5" r="1.2"/>
          </svg>
          <span>{{ isRolling ? '摇骰中...' : '摇骰子' }}</span>
        </span>
      </button>
    </footer>

    <!-- 统计面板 - Bottom Sheet -->
    <Teleport to="body">
      <transition name="sheet-fade">
        <div
          v-if="showStats"
          class="sheet-overlay"
          @click="showStats = false"
        ></div>
      </transition>
      <transition name="sheet-slide">
        <div
          v-if="showStats"
          class="sheet-container"
          @touchstart="onSheetTouchStart"
          @touchmove="onSheetTouchMove"
          @touchend="onSheetTouchEnd"
          :style="sheetTranslateY > 0 ? { transform: `translateY(${sheetTranslateY}px)` } : {}"
        >
          <div class="sheet-handle-bar">
            <div class="sheet-handle"></div>
          </div>
          <div class="sheet-header">
            <h2 class="sheet-title">点数统计</h2>
            <button class="sheet-close" @click="showStats = false">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <path d="M18 6L6 18M6 6l12 12"/>
              </svg>
            </button>
          </div>

          <div class="sheet-body">
            <!-- 概要数据 -->
            <div class="summary-row">
              <div class="summary-card">
                <div class="summary-num">{{ totalRolls }}</div>
                <div class="summary-label">总骰子数</div>
              </div>
              <div class="summary-card">
                <div class="summary-num">{{ rollHistory.length }}</div>
                <div class="summary-label">投掷次数</div>
              </div>
              <div class="summary-card">
                <div class="summary-num">{{ mostCommon }}</div>
                <div class="summary-label">最常见点</div>
              </div>
              <div class="summary-card">
                <div class="summary-num">{{ averageTotal }}</div>
                <div class="summary-label">平均总点</div>
              </div>
            </div>

            <!-- 各点数统计 -->
            <div class="freq-section">
              <div class="freq-title">各点数频率</div>
              <div class="freq-list">
                <div v-for="i in 6" :key="i" class="freq-item">
                  <div class="freq-face">{{ i }}</div>
                  <div class="freq-bar-track">
                    <div
                      class="freq-bar-fill"
                      :style="{ width: `${totalRolls > 0 ? (diceStats[i] / totalRolls) * 100 : 0}%` }"
                    ></div>
                  </div>
                  <div class="freq-count">{{ diceStats[i] }}</div>
                  <div class="freq-pct">
                    {{ totalRolls > 0 ? ((diceStats[i] / totalRolls) * 100).toFixed(1) : '0.0' }}%
                  </div>
                </div>
              </div>
            </div>

            <!-- 操作按钮 -->
            <button class="sheet-clear-btn" @click="clearHistory" :disabled="rollHistory.length === 0">
              清空所有记录
            </button>
          </div>
        </div>
      </transition>
    </Teleport>
  </div>
</template>

<style scoped>
/* ============ 基础布局 ============ */
.dice-game {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: #0a0a14;
  background-image:
    radial-gradient(ellipse at 50% 0%, rgba(102, 126, 234, 0.12) 0%, transparent 60%),
    radial-gradient(ellipse at 80% 100%, rgba(118, 75, 162, 0.08) 0%, transparent 50%);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  color: #fff;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  touch-action: manipulation;
  -webkit-user-select: none;
  user-select: none;
}

/* ============ 顶部栏 ============ */
.top-bar {
  display: flex;
  align-items: center;
  padding: 10px 16px;
  padding-top: max(10px, env(safe-area-inset-top));
  flex-shrink: 0;
  z-index: 10;
}

.icon-btn {
  width: 42px;
  height: 42px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.85);
  cursor: pointer;
  transition: all 0.15s;
  -webkit-tap-highlight-color: transparent;
}

.icon-btn:active {
  transform: scale(0.97);
  background: rgba(255, 255, 255, 0.12);
}

.title {
  flex: 1;
  text-align: center;
  font-size: 1.1rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  background: linear-gradient(135deg, #c4b5fd 0%, #a78bfa 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* ============ 主区域 ============ */
.main-area {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 8px 16px 120px;
  -webkit-overflow-scrolling: touch;
}

/* ============ 骰子数量选择 ============ */
.selector-section {
  margin-bottom: 24px;
}

.selector-label {
  font-size: 0.8rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.4);
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 12px;
  text-align: center;
}

.selector-pills {
  display: flex;
  justify-content: center;
  gap: 10px;
}

/* -- Staggered pill entrance -- */
.pill {
  width: 48px;
  height: 48px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.05);
  border: 1.5px solid rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.5);
  font-size: 1.15rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  -webkit-tap-highlight-color: transparent;
}

.pill.pill-stagger {
  animation: pillStaggerIn 0.45s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}

@keyframes pillStaggerIn {
  0% {
    opacity: 0;
    transform: translateY(18px) scale(0.85);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.pill:active {
  transform: scale(0.97);
}

.pill.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: transparent;
  color: #fff;
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.35);
  transform: scale(1.05);
}

/* ============ 骰子展示区 ============ */
.dice-stage {
  min-height: 140px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  perspective: 600px;
}

.dice-container {
  display: flex;
  justify-content: center;
  gap: 14px;
  flex-wrap: wrap;
  max-width: 100%;
}

.dice-wrapper {
  position: relative;
  width: 68px;
  height: 68px;
  perspective: 400px;
}

/* 3D骰子 */
.dice-3d {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.1s linear;
}

.dice-3d.spinning {
  transition: transform 0.08s linear;
  animation: diceShake3D 0.15s ease-in-out infinite;
}

.dice-3d.landing {
  transition: transform 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  transform: rotateX(0deg) rotateY(0deg) rotateZ(0deg) !important;
}

@keyframes diceShake3D {
  0% { transform: translateY(0px); }
  25% { transform: translateY(-6px) scale(1.03); }
  50% { transform: translateY(-3px); }
  75% { transform: translateY(-8px) scale(1.02); }
  100% { transform: translateY(0px); }
}

.dice-face-front {
  width: 100%;
  height: 100%;
  background: linear-gradient(145deg, #ffffff 0%, #f0f0f0 50%, #e8e8e8 100%);
  border-radius: 16px;
  position: relative;
  box-shadow:
    0 2px 4px rgba(0, 0, 0, 0.2),
    0 8px 24px rgba(0, 0, 0, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.8),
    inset 0 -1px 2px rgba(0, 0, 0, 0.05);
}

.dice-3d.landing .dice-face-front {
  animation: diceBounce 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes diceBounce {
  0% { transform: scale(0.9); }
  50% { transform: scale(1.08); }
  100% { transform: scale(1); }
}

/* -- Enhanced dot with deeper shadow for texture -- */
.dot {
  position: absolute;
  width: 11px;
  height: 11px;
  background: radial-gradient(circle at 35% 35%, #3a3a4a 0%, #1a1a2e 100%);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  box-shadow:
    inset 0 1px 3px rgba(0, 0, 0, 0.45),
    inset 0 -0.5px 1px rgba(255, 255, 255, 0.08),
    0 1px 2px rgba(0, 0, 0, 0.18);
}

/* 骰子阴影 */
.dice-shadow {
  position: absolute;
  bottom: -6px;
  left: 50%;
  transform: translateX(-50%) scaleX(0.6);
  width: 50px;
  height: 8px;
  background: radial-gradient(ellipse, rgba(102, 126, 234, 0.2) 0%, transparent 70%);
  border-radius: 50%;
  opacity: 0;
  transition: opacity 0.3s, transform 0.3s;
}

.dice-shadow.active {
  opacity: 1;
  transform: translateX(-50%) scaleX(1);
}

/* 空状态 */
.empty-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  padding: 20px;
}

.empty-dice-icon {
  opacity: 0.4;
}

.empty-hint {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.25);
}

/* ============ 结果卡片 (enhanced glow + pop) ============ */
.result-card {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(102, 126, 234, 0.12);
  border-radius: 18px;
  padding: 20px;
  margin-bottom: 24px;
  text-align: center;
  backdrop-filter: blur(10px);
  position: relative;
  overflow: hidden;
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.25),
    0 0 40px rgba(102, 126, 234, 0.06);
}

/* background glow for result card */
.result-card::before {
  content: '';
  position: absolute;
  inset: -50%;
  background: radial-gradient(ellipse at center, rgba(102, 126, 234, 0.08) 0%, transparent 60%);
  pointer-events: none;
  animation: resultGlowPulse 2.5s ease-in-out infinite;
}

@keyframes resultGlowPulse {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 1; }
}

.result-top {
  display: flex;
  align-items: baseline;
  justify-content: center;
  gap: 12px;
  margin-bottom: 14px;
  position: relative;
  z-index: 1;
}

.result-label {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.4);
  font-weight: 500;
}

.result-total {
  font-size: 2.8rem;
  font-weight: 800;
  line-height: 1;
  background: linear-gradient(135deg, #667eea 0%, #a78bfa 50%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.result-detail {
  display: flex;
  justify-content: center;
  gap: 8px;
  flex-wrap: wrap;
  position: relative;
  z-index: 1;
}

.result-chip {
  min-width: 38px;
  padding: 6px 14px;
  background: rgba(102, 126, 234, 0.1);
  border: 1px solid rgba(102, 126, 234, 0.15);
  border-radius: 10px;
  font-size: 1rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.8);
}

/* result-pop transition (enhanced) */
.result-pop-enter-active {
  animation: resultIn 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.result-pop-leave-active {
  animation: resultOut 0.2s ease-in;
}
@keyframes resultIn {
  0% { opacity: 0; transform: scale(0.75) translateY(14px); }
  100% { opacity: 1; transform: scale(1) translateY(0); }
}
@keyframes resultOut {
  0% { opacity: 1; transform: scale(1); }
  100% { opacity: 0; transform: scale(0.9); }
}

/* ============ 历史记录 ============ */
.history-section {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.04);
  border-radius: 18px;
  padding: 16px;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.section-title {
  font-size: 0.85rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.5);
}

.section-count {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.05);
  padding: 3px 10px;
  border-radius: 20px;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
  max-height: 240px;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

/* -- Staggered history row entrance -- */
.history-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 12px;
  padding: 10px 12px;
  transition: background 0.2s;
}

.history-row.history-stagger {
  animation: historySlideIn 0.35s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes historySlideIn {
  0% {
    opacity: 0;
    transform: translateX(-12px);
  }
  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

.history-row.latest {
  background: rgba(102, 126, 234, 0.08);
  border: 1px solid rgba(102, 126, 234, 0.12);
}

.history-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.history-index {
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.25);
  min-width: 28px;
}

.history-dice-group {
  display: flex;
  gap: 4px;
}

.history-dice-val {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.06);
  border-radius: 7px;
  font-size: 0.8rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.7);
}

.history-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.history-total {
  font-size: 1.1rem;
  font-weight: 800;
  color: rgba(255, 255, 255, 0.8);
  min-width: 28px;
  text-align: right;
}

.history-time {
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.2);
  font-variant-numeric: tabular-nums;
}

/* ============ 底部操作栏 ============ */
.action-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 10px 16px;
  padding-bottom: max(10px, env(safe-area-inset-bottom));
  background: rgba(10, 10, 20, 0.92);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border-top: 1px solid rgba(255, 255, 255, 0.06);
  display: flex;
  gap: 10px;
  z-index: 100;
}

.btn-reset {
  width: 52px;
  height: 52px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  transition: all 0.15s;
  -webkit-tap-highlight-color: transparent;
}

.btn-reset:active:not(:disabled) {
  transform: scale(0.97);
  background: rgba(255, 255, 255, 0.12);
}

.btn-reset:disabled {
  opacity: 0.3;
}

/* -- Roll button with shimmer -- */
.btn-roll {
  flex: 1;
  height: 52px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  border-radius: 16px;
  color: #fff;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
  transition: all 0.15s;
  -webkit-tap-highlight-color: transparent;
}

.btn-roll:active:not(:disabled) {
  transform: scale(0.97);
  box-shadow: 0 2px 10px rgba(102, 126, 234, 0.4);
}

.btn-roll:disabled {
  opacity: 0.65;
}

.btn-roll:disabled .btn-shimmer {
  display: none;
}

/* Shimmer sweep on roll button */
.btn-shimmer {
  position: absolute;
  top: 0;
  left: -100%;
  width: 60%;
  height: 100%;
  background: linear-gradient(
    105deg,
    transparent 0%,
    transparent 35%,
    rgba(255, 255, 255, 0.3) 50%,
    transparent 65%,
    transparent 100%
  );
  animation: shimmerSweep 3s cubic-bezier(0.22, 1, 0.36, 1) infinite;
  pointer-events: none;
  z-index: 1;
}

@keyframes shimmerSweep {
  0% { left: -100%; }
  40% { left: 160%; }
  100% { left: 160%; }
}

.btn-roll-inner {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 1rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  position: relative;
  z-index: 2;
}

.roll-spinner {
  animation: spin 0.7s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* ============ 统计面板 - Bottom Sheet ============ */
.sheet-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 500;
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
}

.sheet-container {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  max-height: 85vh;
  background: linear-gradient(180deg, #161625 0%, #0e0e1a 100%);
  border-top-left-radius: 24px;
  border-top-right-radius: 24px;
  z-index: 501;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 -8px 40px rgba(0, 0, 0, 0.5);
  padding-bottom: max(16px, env(safe-area-inset-bottom));
  will-change: transform;
}

.sheet-handle-bar {
  display: flex;
  justify-content: center;
  padding: 12px 0 4px;
  flex-shrink: 0;
}

.sheet-handle {
  width: 36px;
  height: 4px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 2px;
}

.sheet-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 20px 16px;
  flex-shrink: 0;
}

.sheet-title {
  font-size: 1.15rem;
  font-weight: 700;
  color: #fff;
}

.sheet-close {
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.06);
  border: none;
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
}

.sheet-close:active {
  background: rgba(255, 255, 255, 0.12);
}

.sheet-body {
  flex: 1;
  overflow-y: auto;
  padding: 0 20px 20px;
  -webkit-overflow-scrolling: touch;
}

/* 概要行 */
.summary-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  margin-bottom: 24px;
}

.summary-card {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 14px;
  padding: 14px 8px;
  text-align: center;
}

.summary-num {
  font-size: 1.3rem;
  font-weight: 800;
  color: #a78bfa;
  line-height: 1.2;
  margin-bottom: 4px;
}

.summary-label {
  font-size: 0.65rem;
  color: rgba(255, 255, 255, 0.35);
  font-weight: 500;
}

/* 频率统计 */
.freq-section {
  margin-bottom: 24px;
}

.freq-title {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.4);
  font-weight: 600;
  margin-bottom: 14px;
  letter-spacing: 0.5px;
}

.freq-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.freq-item {
  display: flex;
  align-items: center;
  gap: 10px;
}

.freq-face {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(102, 126, 234, 0.1);
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 800;
  color: #667eea;
  flex-shrink: 0;
}

.freq-bar-track {
  flex: 1;
  height: 6px;
  background: rgba(255, 255, 255, 0.06);
  border-radius: 3px;
  overflow: hidden;
}

.freq-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  border-radius: 3px;
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  min-width: 0;
}

.freq-count {
  font-size: 0.85rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.7);
  min-width: 24px;
  text-align: right;
}

.freq-pct {
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.3);
  min-width: 40px;
  text-align: right;
  font-variant-numeric: tabular-nums;
}

/* 清空按钮 */
.sheet-clear-btn {
  width: 100%;
  padding: 14px;
  background: rgba(239, 68, 68, 0.08);
  border: 1px solid rgba(239, 68, 68, 0.15);
  border-radius: 14px;
  color: #ef4444;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s;
  -webkit-tap-highlight-color: transparent;
}

.sheet-clear-btn:active:not(:disabled) {
  background: rgba(239, 68, 68, 0.15);
  transform: scale(0.97);
}

.sheet-clear-btn:disabled {
  opacity: 0.3;
}

/* sheet transitions */
.sheet-fade-enter-active,
.sheet-fade-leave-active {
  transition: opacity 0.3s ease;
}
.sheet-fade-enter-from,
.sheet-fade-leave-to {
  opacity: 0;
}

.sheet-slide-enter-active {
  transition: transform 0.35s cubic-bezier(0.32, 0.72, 0, 1);
}
.sheet-slide-leave-active {
  transition: transform 0.25s cubic-bezier(0.32, 0.72, 0, 1);
}
.sheet-slide-enter-from,
.sheet-slide-leave-to {
  transform: translateY(100%);
}

/* ============ 滚动条美化 ============ */
.main-area::-webkit-scrollbar,
.history-list::-webkit-scrollbar,
.sheet-body::-webkit-scrollbar {
  width: 3px;
}
.main-area::-webkit-scrollbar-track,
.history-list::-webkit-scrollbar-track,
.sheet-body::-webkit-scrollbar-track {
  background: transparent;
}
.main-area::-webkit-scrollbar-thumb,
.history-list::-webkit-scrollbar-thumb,
.sheet-body::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 3px;
}
</style>
