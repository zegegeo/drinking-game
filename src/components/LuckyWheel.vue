<script setup>
import { ref, computed } from 'vue'

defineEmits(['goBack'])

const options = ref(['喝酒一杯', '真心话', '大冒险', '表演才艺', '喝半杯', '免死金牌'])
const newOption = ref('')
const showAddPanel = ref(false)
const isSpinning = ref(false)
const rotation = ref(0)
const showResult = ref(false)
const winnerIndex = ref(-1)

const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(25)
  }
}

/* 交替柔和配色：深/浅两色交替 */
const segColors = [
  ['#2d1b69', '#3b2080'],  // 深紫
  ['#1e1145', '#2a1660'],  // 暗紫
]

const colors = [
  '#667eea', '#764ba2',
  '#f093fb', '#f5576c',
  '#4facfe', '#00f2fe',
  '#43e97b', '#38f9d7',
  '#fa709a', '#fee140',
  '#a18cd1', '#fbc2eb'
]

const segmentAngle = computed(() => {
  return 360 / options.value.length
})

const wheelStyle = computed(() => {
  const segments = options.value.map((_, i) => {
    const startAngle = i * segmentAngle.value
    const endAngle = (i + 1) * segmentAngle.value
    const c = segColors[i % 2]
    return `${c[0]} ${startAngle}deg, ${c[1]} ${endAngle}deg`
  }).join(', ')

  return {
    background: `conic-gradient(${segments})`,
    transform: `rotate(${rotation.value}deg)`
  }
})

const addOption = () => {
  if (newOption.value.trim() && !isSpinning.value) {
    triggerHaptic()
    options.value.push(newOption.value.trim())
    newOption.value = ''
  }
}

const removeOption = (index) => {
  if (!isSpinning.value && options.value.length > 2) {
    triggerHaptic()
    options.value.splice(index, 1)
  }
}

const spin = () => {
  if (isSpinning.value || options.value.length < 2) return
  triggerHaptic()

  isSpinning.value = true
  showResult.value = false

  const spins = 5 + Math.random() * 3
  const randomOffset = Math.random() * 360
  const targetRotation = rotation.value + (spins * 360) + randomOffset

  const duration = 4000
  const startRotation = rotation.value
  const startTime = Date.now()

  const animate = () => {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)

    const easeOut = 1 - Math.pow(1 - progress, 3)
    rotation.value = startRotation + (targetRotation - startRotation) * easeOut

    if (progress < 1) {
      requestAnimationFrame(animate)
    } else {
      isSpinning.value = false
      showResult.value = true

      const normalizedRotation = rotation.value % 360
      winnerIndex.value = Math.floor((360 - normalizedRotation) / segmentAngle.value) % options.value.length

      triggerHaptic()
      if ('vibrate' in navigator) {
        navigator.vibrate([50, 50, 50])
      }
    }
  }

  animate()
}

const reset = () => {
  triggerHaptic()
  rotation.value = 0
  showResult.value = false
  winnerIndex.value = -1
}

const getLabelStyle = (index) => {
  const angle = (index + 0.5) * segmentAngle.value - 90
  const radian = (angle * Math.PI) / 180
  const radius = 38

  /* 给每个标签分配一个柔和的主题色 */
  const labelColors = ['#c4b5fd', '#93c5fd', '#f9a8d4', '#86efac', '#fcd34d', '#a5b4fc', '#fca5a5', '#67e8f9', '#d8b4fe', '#fde68a']

  return {
    left: `${50 + radius * Math.cos(radian)}%`,
    top: `${50 + radius * Math.sin(radian)}%`,
    transform: `translate(-50%, -50%) rotate(${angle + 90}deg)`,
    color: labelColors[index % labelColors.length]
  }
}

const toggleAddPanel = () => {
  showAddPanel.value = !showAddPanel.value
}

const closePanel = () => {
  showAddPanel.value = false
}
</script>

<template>
  <div class="lucky-wheel-mobile">
    <!-- 背景装饰 -->
    <div class="lw-bg">
      <div class="lw-mesh lw-mesh-1"></div>
      <div class="lw-mesh lw-mesh-2"></div>
      <div class="lw-mesh lw-mesh-3"></div>
    </div>
    <div class="lw-grain"></div>

    <!-- 顶部标题栏 -->
    <header class="mobile-header">
      <button class="back-btn" @click="$emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title">幸运大转盘</h1>
      <button class="add-btn" @click="toggleAddPanel">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M12 5v14M5 12h14"/>
        </svg>
      </button>
    </header>

    <!-- 主游戏区域 -->
    <main class="game-area">
      <!-- 转盘 -->
      <div class="wheel-container" :class="{ spinning: isSpinning }">
        <!-- 外圈渐变光环 -->
        <div class="wheel-outer-glow"></div>
        <div class="wheel-outer-ring"></div>
        <!-- 指针 -->
        <div class="wheel-pointer">
          <div class="pointer-arrow"></div>
        </div>
        <!-- 转盘主体 -->
        <div class="wheel" :style="wheelStyle">
          <!-- 标签 -->
          <div
            v-for="(opt, index) in options"
            :key="'label-' + index"
            class="wheel-label"
            :style="getLabelStyle(index)"
          >
            {{ opt }}
          </div>
        </div>
        <!-- 中心按钮 -->
        <div class="wheel-center">
          <div class="center-ring" :class="{ active: !isSpinning }"></div>
          <div class="center-btn" @click="spin" :class="{ disabled: isSpinning }">
            <span v-if="!isSpinning">GO</span>
            <span v-else class="spinning-dots">
              <span class="dot"></span>
              <span class="dot"></span>
              <span class="dot"></span>
            </span>
          </div>
        </div>
      </div>

      <!-- 结果展示 -->
      <Transition name="result-pop">
        <div v-if="showResult && winnerIndex >= 0" class="result-display">
          <div class="result-card">
            <div class="result-glow"></div>
            <div class="result-confetti">
              <span v-for="i in 6" :key="i" class="confetti-piece" :style="{ '--ci': i }"></span>
            </div>
            <div class="result-emoji">🎉</div>
            <div class="result-label">恭喜选中</div>
            <div class="result-value">{{ options[winnerIndex] }}</div>
            <button class="reset-btn" @click="reset">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M1 4v6h6M23 20v-6h-6"/>
                <path d="M20.49 9A9 9 0 0 0 5.64 5.64L1 10m22 4l-4.64 4.36A9 9 0 0 1 3.51 15"/>
              </svg>
              再来一次
            </button>
          </div>
        </div>
      </Transition>
    </main>

    <!-- 底部弹出面板 (Bottom Sheet) -->
    <Transition name="sheet">
      <div v-if="showAddPanel" class="sheet-overlay" @click.self="closePanel">
        <div class="bottom-sheet">
          <div class="sheet-handle" @click="closePanel">
            <div class="handle-bar"></div>
          </div>
          <div class="sheet-header">
            <h2 class="sheet-title">管理选项</h2>
            <span class="sheet-count">{{ options.length }} 项</span>
          </div>
          <div class="add-input">
            <input
              v-model="newOption"
              type="text"
              placeholder="输入选项内容..."
              @keypress.enter="addOption"
              :disabled="isSpinning"
            />
            <button class="confirm-btn" @click="addOption" :disabled="!newOption.trim()">
              添加
            </button>
          </div>
          <div class="options-list">
            <div
              v-for="(opt, index) in options"
              :key="index"
              class="option-item option-stagger"
              :style="{ animationDelay: `${index * 0.04}s` }"
              @click="removeOption(index)"
            >
              <span class="option-dot" :style="{ background: colors[index % colors.length] }"></span>
              <span class="option-text">{{ opt }}</span>
              <svg v-if="options.length > 2" class="remove-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M18 6L6 18M6 6l12 12"/>
              </svg>
            </div>
          </div>
          <div class="panel-tip">点击选项可以删除（至少保留2个）</div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
/* ========== 全局容器 ========== */
.lucky-wheel-mobile {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: linear-gradient(160deg, #0f0c29 0%, #1a1040 45%, #120e2a 100%);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  color: #fff;
}

/* ========== 背景网格渐变 ========== */
.lw-bg {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  pointer-events: none;
  z-index: 0;
}

.lw-mesh {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
}

.lw-mesh-1 {
  background:
    radial-gradient(ellipse 55% 50% at 20% 15%, rgba(255,107,107,0.15), transparent),
    radial-gradient(ellipse 50% 40% at 80% 20%, rgba(255,215,0,0.12), transparent);
  animation: lwDrift1 20s ease-in-out infinite alternate;
}

.lw-mesh-2 {
  background:
    radial-gradient(ellipse 45% 55% at 70% 65%, rgba(102,126,234,0.12), transparent),
    radial-gradient(ellipse 40% 35% at 25% 80%, rgba(250,112,154,0.08), transparent);
  animation: lwDrift2 24s ease-in-out infinite alternate;
}

.lw-mesh-3 {
  background:
    radial-gradient(ellipse 35% 30% at 50% 45%, rgba(255,142,83,0.06), transparent);
  animation: lwDrift3 18s ease-in-out infinite alternate;
}

@keyframes lwDrift1 {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(18px, -12px) scale(1.04); }
}
@keyframes lwDrift2 {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(-14px, 18px) scale(1.06); }
}
@keyframes lwDrift3 {
  0% { transform: translate(0, 0); opacity: 0.8; }
  100% { transform: translate(10px, 8px); opacity: 1; }
}

/* ========== 噪点 ========== */
.lw-grain {
  position: fixed;
  top: -50%; left: -50%;
  width: 200%; height: 200%;
  pointer-events: none;
  z-index: 1;
  opacity: 0.022;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  background-size: 128px 128px;
}

/* ========== Header ========== */
.mobile-header {
  display: flex;
  align-items: center;
  padding: 10px 16px;
  padding-top: max(10px, env(safe-area-inset-top));
  background: rgba(15, 12, 41, 0.6);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  position: relative;
  z-index: 20;
}

.back-btn, .add-btn {
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.9);
  cursor: pointer;
  transition: all 0.15s ease;
}

.back-btn:active, .add-btn:active {
  background: rgba(255, 255, 255, 0.1);
  transform: scale(0.97);
}

.header-title {
  flex: 1;
  text-align: center;
  font-size: 1.1rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  background: linear-gradient(135deg, #c4b5fd 0%, #a78bfa 50%, #818cf8 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* ========== 游戏区域 ========== */
.game-area {
  flex: 1;
  overflow-y: auto;
  padding: 16px 16px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  z-index: 2;
  padding-bottom: max(32px, env(safe-area-inset-bottom));
}

/* ========== 转盘 ========== */
.wheel-container {
  position: relative;
  width: 300px;
  height: 300px;
  margin: 8px 0 16px;
  flex-shrink: 0;
  animation: wheelEnter 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes wheelEnter {
  from { opacity: 0; transform: scale(0.85); }
  to { opacity: 1; transform: scale(1); }
}

/* 外圈柔和光晕 */
.wheel-outer-glow {
  position: absolute;
  top: -20px; left: -20px; right: -20px; bottom: -20px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(167,139,250,0.12), rgba(99,102,241,0.06), transparent 70%);
  filter: blur(8px);
  pointer-events: none;
  z-index: 0;
  transition: opacity 0.3s ease;
}

.wheel-container.spinning .wheel-outer-glow {
  animation: glowPulse 0.8s ease-in-out infinite alternate;
}

@keyframes glowPulse {
  from { opacity: 1; filter: blur(8px); }
  to { opacity: 1.3; filter: blur(14px); }
}

/* 外圈装饰环 */
.wheel-outer-ring {
  position: absolute;
  top: -8px; left: -8px; right: -8px; bottom: -8px;
  border-radius: 50%;
  border: 2px solid rgba(167, 139, 250, 0.15);
  background: transparent;
  z-index: 1;
  box-shadow: 0 0 20px rgba(167, 139, 250, 0.06);
}

/* 指针 — 简洁三角 */
.wheel-pointer {
  position: absolute;
  top: -14px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
}

.pointer-arrow {
  width: 28px;
  height: 28px;
  background: linear-gradient(180deg, #c4b5fd, #a78bfa);
  clip-path: polygon(50% 100%, 10% 0%, 90% 0%);
  filter: drop-shadow(0 3px 8px rgba(167, 139, 250, 0.5));
}

/* 转盘主体 */
.wheel {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  position: relative;
  box-shadow:
    0 0 0 3px rgba(167, 139, 250, 0.12),
    0 12px 40px rgba(0, 0, 0, 0.4),
    inset 0 0 60px rgba(0, 0, 0, 0.3);
  z-index: 2;
  overflow: hidden;
}

.wheel-label {
  position: absolute;
  font-size: 0.75rem;
  font-weight: 700;
  text-shadow: 0 1px 6px rgba(0, 0, 0, 0.8);
  white-space: nowrap;
  max-width: 58px;
  overflow: hidden;
  text-overflow: ellipsis;
  text-align: center;
  pointer-events: none;
  z-index: 3;
  letter-spacing: 0.5px;
}

/* 中心按钮 */
.wheel-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 5;
}

.center-ring {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  width: 84px;
  height: 84px;
  border-radius: 50%;
  box-shadow: 0 0 16px 3px rgba(167, 139, 250, 0.15);
  pointer-events: none;
  transition: box-shadow 0.3s ease;
}

.center-ring.active {
  animation: ringPulse 2.2s ease-in-out infinite;
}

@keyframes ringPulse {
  0%, 100% {
    box-shadow: 0 0 14px 2px rgba(167, 139, 250, 0.12);
    transform: translate(-50%, -50%) scale(1);
  }
  50% {
    box-shadow: 0 0 24px 6px rgba(167, 139, 250, 0.25);
    transform: translate(-50%, -50%) scale(1.04);
  }
}

.center-btn {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background: linear-gradient(145deg, #2d1b69 0%, #1a1040 100%);
  border: 2px solid rgba(167, 139, 250, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.1rem;
  font-weight: 800;
  color: #c4b5fd;
  cursor: pointer;
  box-shadow:
    0 4px 20px rgba(0, 0, 0, 0.5),
    inset 0 1px 1px rgba(167, 139, 250, 0.15);
  transition: all 0.15s ease;
  position: relative;
  z-index: 1;
  letter-spacing: 2px;
}

.center-btn:active:not(.disabled) {
  transform: scale(0.95);
  border-color: rgba(167, 139, 250, 0.5);
  box-shadow:
    0 2px 12px rgba(0, 0, 0, 0.6),
    0 0 16px rgba(167, 139, 250, 0.2);
}

.center-btn.disabled {
  opacity: 0.75;
  cursor: not-allowed;
}

.spinning-dots {
  display: flex;
  gap: 4px;
  align-items: center;
}

.spinning-dots .dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #fff;
  animation: dotBounce 0.6s ease-in-out infinite;
}

.spinning-dots .dot:nth-child(2) { animation-delay: 0.1s; }
.spinning-dots .dot:nth-child(3) { animation-delay: 0.2s; }

@keyframes dotBounce {
  0%, 100% { opacity: 0.3; transform: translateY(0); }
  50% { opacity: 1; transform: translateY(-4px); }
}

/* ========== 结果展示 ========== */
.result-display {
  margin-top: 20px;
  width: 100%;
  max-width: 340px;
  display: flex;
  justify-content: center;
}

.result-card {
  position: relative;
  width: 100%;
  text-align: center;
  padding: 26px 22px;
  background: rgba(15, 12, 41, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  overflow: hidden;
}

.result-glow {
  position: absolute;
  top: -50%; left: -50%;
  width: 200%; height: 200%;
  background: radial-gradient(ellipse at center, rgba(167, 139, 250, 0.08) 0%, transparent 60%);
  pointer-events: none;
  animation: resultGlowPulse 2s ease-in-out infinite;
}

@keyframes resultGlowPulse {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 1; }
}

/* 彩屑 */
.result-confetti {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  pointer-events: none;
  overflow: hidden;
}

.confetti-piece {
  position: absolute;
  width: 8px;
  height: 8px;
  border-radius: 2px;
  opacity: 0;
  top: -8px;
  animation: confettiFall 2.2s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}

.confetti-piece:nth-child(1) { left: 12%; background: #ff6b6b; animation-delay: 0s; width: 7px; height: 10px; }
.confetti-piece:nth-child(2) { left: 30%; background: #ffd700; animation-delay: 0.12s; width: 6px; height: 6px; border-radius: 50%; }
.confetti-piece:nth-child(3) { left: 50%; background: #4facfe; animation-delay: 0.2s; width: 9px; height: 7px; }
.confetti-piece:nth-child(4) { left: 68%; background: #43e97b; animation-delay: 0.08s; width: 6px; height: 9px; border-radius: 50%; }
.confetti-piece:nth-child(5) { left: 82%; background: #f093fb; animation-delay: 0.28s; width: 8px; height: 6px; }
.confetti-piece:nth-child(6) { left: 92%; background: #fee140; animation-delay: 0.16s; width: 5px; height: 8px; }

@keyframes confettiFall {
  0% { opacity: 1; transform: translateY(0) rotate(0deg) scale(1); }
  30% { opacity: 1; }
  100% { opacity: 0; transform: translateY(200px) rotate(360deg) scale(0.4); }
}

.result-emoji {
  font-size: 2rem;
  margin-bottom: 8px;
  position: relative;
  z-index: 1;
  animation: emojiPop 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes emojiPop {
  from { transform: scale(0); }
  to { transform: scale(1); }
}

.result-label {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.45);
  margin-bottom: 10px;
  position: relative;
  z-index: 1;
}

.result-value {
  font-size: 1.9rem;
  font-weight: 700;
  background: linear-gradient(135deg, #c4b5fd 0%, #a78bfa 50%, #818cf8 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 18px;
  position: relative;
  z-index: 1;
}

.result-display .reset-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 11px 28px;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  color: rgba(255, 255, 255, 0.85);
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s ease;
  position: relative;
  z-index: 1;
}

.reset-btn:active {
  background: rgba(255, 255, 255, 0.12);
  transform: scale(0.97);
}

/* 结果弹出动画 */
.result-pop-enter-active {
  animation: resultBounceIn 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.result-pop-leave-active {
  animation: resultFadeOut 0.25s ease-out forwards;
}

@keyframes resultBounceIn {
  0% { opacity: 0; transform: scale(0.5) translateY(30px); }
  60% { opacity: 1; transform: scale(1.03) translateY(-4px); }
  100% { opacity: 1; transform: scale(1) translateY(0); }
}

@keyframes resultFadeOut {
  to { opacity: 0; transform: scale(0.95) translateY(10px); }
}

/* ========== Bottom Sheet ========== */
.sheet-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.55);
  z-index: 100;
  display: flex;
  align-items: flex-end;
}

.bottom-sheet {
  width: 100%;
  max-height: 65vh;
  background: rgba(20, 16, 48, 0.95);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-top-left-radius: 24px;
  border-top-right-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-bottom: none;
  padding: 0 20px;
  padding-bottom: max(24px, env(safe-area-inset-bottom));
  display: flex;
  flex-direction: column;
}

.sheet-handle {
  display: flex;
  justify-content: center;
  padding: 12px 0 6px;
  cursor: pointer;
}

.handle-bar {
  width: 40px;
  height: 4px;
  border-radius: 2px;
  background: rgba(255, 255, 255, 0.15);
}

.sheet-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 0 16px;
}

.sheet-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
}

.sheet-count {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.4);
  background: rgba(255, 255, 255, 0.06);
  padding: 4px 12px;
  border-radius: 20px;
}

/* Sheet 内的输入 */
.add-input {
  display: flex;
  gap: 10px;
  margin-bottom: 16px;
}

.add-input input {
  flex: 1;
  padding: 14px 16px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: #fff;
  font-size: 0.95rem;
  outline: none;
  transition: border-color 0.2s ease;
}

.add-input input:focus {
  border-color: rgba(167, 139, 250, 0.35);
}

.add-input input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

.add-input input:disabled {
  opacity: 0.5;
}

.confirm-btn {
  padding: 14px 22px;
  background: linear-gradient(135deg, #a78bfa 0%, #818cf8 100%);
  border: none;
  border-radius: 14px;
  color: #fff;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s ease;
  white-space: nowrap;
}

.confirm-btn:active {
  transform: scale(0.97);
}

.confirm-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

/* Sheet 内的选项列表 */
.options-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  overflow-y: auto;
  flex: 1;
  min-height: 0;
  max-height: 35vh;
  padding-right: 4px;
}

.options-list::-webkit-scrollbar { width: 3px; }
.options-list::-webkit-scrollbar-track { background: transparent; }
.options-list::-webkit-scrollbar-thumb { background: rgba(255, 255, 255, 0.1); border-radius: 3px; }

.option-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 14px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 14px;
  cursor: pointer;
  transition: all 0.15s ease;
  flex-shrink: 0;
}

.option-item.option-stagger {
  animation: optionSlideIn 0.35s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes optionSlideIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

.option-item:active {
  background: rgba(255, 107, 107, 0.1);
  border-color: rgba(255, 107, 107, 0.18);
  transform: scale(0.97);
}

.option-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  flex-shrink: 0;
  box-shadow: 0 0 6px rgba(255, 255, 255, 0.1);
}

.option-text {
  flex: 1;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.8);
}

.remove-icon {
  color: rgba(255, 107, 107, 0.45);
  flex-shrink: 0;
  transition: color 0.2s ease;
}

.option-item:active .remove-icon {
  color: rgba(255, 107, 107, 0.9);
}

.panel-tip {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.25);
  text-align: center;
  margin-top: 14px;
  padding-bottom: 4px;
}

/* Sheet 过渡动画 */
.sheet-enter-active { transition: opacity 0.3s ease; }
.sheet-enter-active .bottom-sheet { animation: sheetSlideUp 0.35s cubic-bezier(0.22, 1, 0.36, 1); }
.sheet-leave-active { transition: opacity 0.25s ease; }
.sheet-leave-active .bottom-sheet { animation: sheetSlideDown 0.25s ease-in forwards; }
.sheet-enter-from, .sheet-leave-to { opacity: 0; }

@keyframes sheetSlideUp {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}

@keyframes sheetSlideDown {
  from { transform: translateY(0); }
  to { transform: translateY(100%); }
}

/* ========== 小屏适配 ========== */
@media (max-height: 700px) {
  .wheel-container { width: 255px; height: 255px; }
  .wheel-outer-ring { top: -7px; left: -7px; right: -7px; bottom: -7px; }
  .center-btn { width: 56px; height: 56px; font-size: 1rem; }
  .center-ring { width: 72px; height: 72px; }
  .wheel-label { font-size: 0.68rem; max-width: 50px; }
  .result-value { font-size: 1.5rem; }
}

@media (max-height: 600px) {
  .wheel-container { width: 215px; height: 215px; margin: 4px 0 8px; }
  .game-area { padding: 8px 16px; }
  .center-btn { width: 48px; height: 48px; font-size: 0.9rem; }
  .center-ring { width: 62px; height: 62px; }
}

/* 大屏优化 */
@media (min-width: 768px) {
  .wheel-container { width: 340px; height: 340px; }
  .center-btn { width: 72px; height: 72px; font-size: 1.2rem; }
  .center-ring { width: 92px; height: 92px; }
  .bottom-sheet {
    max-width: 480px;
    margin: 0 auto;
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
  }
}
</style>
