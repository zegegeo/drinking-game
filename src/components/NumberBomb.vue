<script setup>
import { ref, computed } from 'vue'

defineEmits(['goBack'])

// 游戏阶段: setup / playing / exploded
const phase = ref('setup')

// 设置阶段
const playerCount = ref(0)
const playerOptions = [2, 3, 4, 5, 6, 7, 8, 9, 10]

// 游戏状态
const bombNumber = ref(0)
const currentMin = ref(1)
const currentMax = ref(100)
const currentPlayerIndex = ref(0)
const guessInput = ref('')
const guessHistory = ref([])
const errorMessage = ref('')

// 触觉反馈
const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(25)
  }
}

// 计算属性
const players = computed(() => {
  return Array.from({ length: playerCount.value }, (_, i) => `玩家${i + 1}`)
})

const currentPlayer = computed(() => players.value[currentPlayerIndex.value])

const rangeSize = computed(() => currentMax.value - currentMin.value)

// 开始游戏
function startGame() {
  // 随机生成炸弹数字 (2 ~ 99)
  bombNumber.value = Math.floor(Math.random() * 98) + 2
  currentMin.value = 1
  currentMax.value = 100
  currentPlayerIndex.value = 0
  guessHistory.value = []
  errorMessage.value = ''
  phase.value = 'playing'
  triggerHaptic()
}

// 猜数字
function makeGuess() {
  const guess = parseInt(guessInput.value)
  if (isNaN(guess)) {
    errorMessage.value = '请输入有效的数字'
    return
  }
  if (guess <= currentMin.value || guess >= currentMax.value) {
    errorMessage.value = `请输入 ${currentMin.value + 1} 到 ${currentMax.value - 1} 之间的数字`
    return
  }

  errorMessage.value = ''
  triggerHaptic()

  // 记录猜测
  guessHistory.value.unshift({
    player: currentPlayer.value,
    guess,
    hit: guess === bombNumber.value
  })
  if (guessHistory.value.length > 12) guessHistory.value.pop()

  guessInput.value = ''

  // 判断是否踩雷
  if (guess === bombNumber.value) {
    phase.value = 'exploded'
    return
  }

  // 缩小范围
  if (guess < bombNumber.value) {
    currentMin.value = guess
  } else {
    currentMax.value = guess
  }

  // 下一个玩家
  currentPlayerIndex.value = (currentPlayerIndex.value + 1) % playerCount.value
}

// 再来一局（保持人数）
function playAgain() {
  startGame()
}

// 换人玩（回到设置）
function backToSetup() {
  phase.value = 'setup'
  playerCount.value = 0
}
</script>

<template>
  <div class="number-bomb-mobile" :class="{ 'screen-shake': phase === 'exploded' }">
    <!-- 背景红色光晕装饰 -->
    <div class="bg-decoration">
      <div class="bg-orb bg-orb-1"></div>
      <div class="bg-orb bg-orb-2"></div>
      <div class="bg-orb bg-orb-3"></div>
    </div>

    <!-- 爆炸闪红效果 -->
    <div v-if="phase === 'exploded'" class="explosion-flash"></div>

    <!-- 顶部标题栏 -->
    <header class="mobile-header">
      <button class="back-btn" @click="phase === 'setup' ? $emit('goBack') : backToSetup()">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title">💣 数字炸弹</h1>
      <div class="header-spacer"></div>
    </header>

    <!-- 设置阶段：选择人数 -->
    <main v-if="phase === 'setup'" class="game-area">
      <div class="lobby-intro">
        <div class="intro-emoji">💣</div>
        <div class="intro-text">猜中炸弹数字的人喝酒！</div>
      </div>

      <div class="setup-section">
        <div class="section-label">选择玩家数量</div>
        <div class="player-pills">
          <button
            v-for="(n, idx) in playerOptions"
            :key="n"
            class="pill pill-stagger"
            :class="{ active: playerCount === n }"
            :style="{ animationDelay: (idx * 0.05) + 's' }"
            @click="playerCount = n; triggerHaptic()"
          >
            {{ n }}人
          </button>
        </div>
      </div>

      <div v-if="playerCount > 0" class="player-preview">
        <div class="section-label">参与玩家</div>
        <div class="preview-chips">
          <span
            v-for="(p, idx) in players"
            :key="p"
            class="preview-chip"
            :style="{ animationDelay: (idx * 0.06) + 's' }"
          >{{ p }}</span>
        </div>
      </div>

      <div class="setup-actions">
        <button
          v-if="playerCount >= 2"
          class="start-btn shimmer-btn"
          @click="startGame"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polygon points="5 3 19 12 5 21 5 3"/>
          </svg>
          开始游戏
        </button>
      </div>

      <div class="rules-hint">
        <div class="rule-item">🎯 系统随机选一个 1~100 的炸弹数字</div>
        <div class="rule-item">🔢 轮流猜数字，范围逐渐缩小</div>
        <div class="rule-item">💥 猜中炸弹数字的人喝酒！</div>
      </div>
    </main>

    <!-- 游戏进行中 -->
    <main v-else-if="phase === 'playing'" class="game-area game-playing">
      <div class="game-header">
        <div class="turn-indicator">
          <span class="turn-label">当前回合</span>
          <span class="turn-player">{{ currentPlayer }}</span>
        </div>
      </div>

      <div class="range-display">
        <div class="range-number min">{{ currentMin }}</div>
        <div class="range-center">
          <div class="bomb-icon" :class="{ 'danger-zone': rangeSize <= 10 }">💣</div>
          <div class="range-label">猜数字</div>
        </div>
        <div class="range-number max">{{ currentMax }}</div>
      </div>

      <div class="range-progress">
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: `${(rangeSize / 99) * 100}%` }"></div>
        </div>
        <div class="progress-text">剩余 {{ rangeSize }} 个数字</div>
      </div>

      <div class="guess-section">
        <input
          v-model.number="guessInput"
          type="number"
          :placeholder="`输入 ${currentMin + 1} ~ ${currentMax - 1}`"
          @keyup.enter="makeGuess"
        >
        <button class="guess-btn" @click="makeGuess" :disabled="!guessInput">
          确认
        </button>
      </div>

      <div v-if="errorMessage" class="error-toast">{{ errorMessage }}</div>

      <!-- 玩家列表 -->
      <div class="mini-players">
        <div
          v-for="(p, idx) in players"
          :key="p"
          class="mini-player"
          :class="{ active: idx === currentPlayerIndex }"
        >
          {{ p }}
        </div>
      </div>

      <div v-if="guessHistory.length > 0" class="history-section">
        <div class="section-label">猜测记录</div>
        <div class="history-list">
          <div v-for="(h, index) in guessHistory" :key="index" class="history-row">
            <span class="history-player">{{ h.player }}</span>
            <span class="history-guess">{{ h.guess }}</span>
          </div>
        </div>
      </div>
    </main>

    <!-- 爆炸结果 -->
    <div v-if="phase === 'exploded'" class="explosion-overlay">
      <div class="explosion-content">
        <div class="boom-icon">💥</div>
        <h2 class="boom-title">BOOM!</h2>
        <p class="boom-player">{{ currentPlayer }} 踩雷了！</p>
        <div class="boom-number">
          炸弹数字 <span class="bomb-value">{{ bombNumber }}</span>
        </div>
        <p class="boom-msg">🍺 喝一杯！</p>
        <div class="result-actions">
          <button class="restart-btn shimmer-btn" @click="playAgain">再来一局</button>
          <button class="reset-btn" @click="backToSetup">换人玩</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.number-bomb-mobile {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: linear-gradient(180deg, #0f0f1a 0%, #1a1030 50%, #0f0f1a 100%);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* ===== 背景红色光晕装饰 ===== */
.bg-decoration {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
}

.bg-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.3;
  animation: orbFloat 14s ease-in-out infinite;
}

.bg-orb-1 {
  width: 180px; height: 180px;
  background: radial-gradient(circle, rgba(255, 107, 107, 0.2), transparent 70%);
  top: 8%; left: -5%;
  animation-delay: 0s;
}
.bg-orb-2 {
  width: 150px; height: 150px;
  background: radial-gradient(circle, rgba(255, 142, 83, 0.15), transparent 70%);
  top: 55%; right: -8%;
  animation-delay: -5s;
}
.bg-orb-3 {
  width: 120px; height: 120px;
  background: radial-gradient(circle, rgba(255, 71, 87, 0.12), transparent 70%);
  bottom: 12%; left: 25%;
  animation-delay: -9s;
}

@keyframes orbFloat {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(12px, -18px) scale(1.05); }
  66% { transform: translate(-8px, 12px) scale(0.95); }
}

/* ===== 爆炸闪红 + 震动 ===== */
.explosion-flash {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(255, 40, 40, 0.15);
  z-index: 150;
  pointer-events: none;
  animation: flashRed 0.8s ease-out forwards;
}

@keyframes flashRed {
  0% { opacity: 1; background: rgba(255, 40, 40, 0.35); }
  30% { opacity: 0.7; background: rgba(255, 40, 40, 0.2); }
  60% { opacity: 0.3; }
  100% { opacity: 0; }
}

.screen-shake {
  animation: screenShake 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes screenShake {
  0% { transform: translate(0, 0); }
  10% { transform: translate(-6px, 3px); }
  20% { transform: translate(5px, -4px); }
  30% { transform: translate(-4px, 2px); }
  40% { transform: translate(3px, -2px); }
  50% { transform: translate(-2px, 1px); }
  60% { transform: translate(1px, -1px); }
  70% { transform: translate(-1px, 0); }
  100% { transform: translate(0, 0); }
}

/* ===== Header ===== */
.mobile-header {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  padding-top: max(12px, env(safe-area-inset-top));
  position: relative;
  z-index: 10;
}

.back-btn {
  width: 44px; height: 44px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 14px;
  color: rgba(255,255,255,0.9);
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.back-btn:active { background: rgba(255,255,255,0.12); transform: scale(0.95); }

.header-title {
  flex: 1; text-align: center;
  font-size: 1.1rem; font-weight: 700;
  background: linear-gradient(135deg, #ff6b6b 0%, #ff8e53 50%, #ff4757 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
.header-spacer { width: 44px; }

.game-area {
  flex: 1; overflow-y: auto;
  padding: 20px 16px 40px;
  position: relative;
  z-index: 1;
}

/* ===== 设置阶段 ===== */
.lobby-intro {
  text-align: center; margin-bottom: 28px;
  animation: introFadeIn 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes introFadeIn {
  from { opacity: 0; transform: translateY(16px); }
  to { opacity: 1; transform: translateY(0); }
}

.intro-emoji { font-size: 3.5rem; margin-bottom: 10px; }
.intro-text { font-size: 1.1rem; color: rgba(255,255,255,0.7); font-weight: 600; }

.setup-section {
  margin-bottom: 24px;
  animation: cardSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.1s both;
}

.section-label {
  font-size: 0.82rem; color: rgba(255,255,255,0.45);
  margin-bottom: 12px; font-weight: 500;
}

.player-pills {
  display: flex; flex-wrap: wrap; gap: 10px;
}

.pill {
  padding: 10px 20px;
  background: rgba(255,255,255,0.05);
  border: 1.5px solid rgba(255,255,255,0.08);
  border-radius: 20px;
  color: rgba(255,255,255,0.7);
  font-size: 0.92rem; font-weight: 500;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}

.pill:active { transform: scale(0.95); }

.pill.active {
  background: linear-gradient(135deg, rgba(255,107,107,0.2), rgba(255,142,83,0.2));
  border-color: rgba(255,107,107,0.5);
  color: #ff6b6b;
  font-weight: 600;
  box-shadow: 0 0 16px rgba(255,107,107,0.15);
}

.pill-stagger {
  animation: pillStaggerIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes pillStaggerIn {
  from { opacity: 0; transform: translateY(10px) scale(0.9); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}

/* 玩家预览 */
.player-preview {
  margin-bottom: 24px;
  animation: cardSlideIn 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.preview-chips {
  display: flex; flex-wrap: wrap; gap: 8px;
}

.preview-chip {
  padding: 8px 16px;
  background: rgba(255,107,107,0.08);
  border: 1px solid rgba(255,107,107,0.15);
  border-radius: 12px;
  font-size: 0.85rem; color: rgba(255,255,255,0.8); font-weight: 500;
  animation: chipFadeIn 0.3s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes chipFadeIn {
  from { opacity: 0; transform: scale(0.85); }
  to { opacity: 1; transform: scale(1); }
}

.setup-actions {
  text-align: center; margin-bottom: 28px;
}

.start-btn {
  width: 100%; max-width: 280px; height: 54px;
  display: inline-flex; align-items: center; justify-content: center; gap: 10px;
  background: linear-gradient(135deg, #ff6b6b, #ff8e53);
  border: none; border-radius: 16px; color: #fff;
  font-size: 1.05rem; font-weight: 600; cursor: pointer;
  box-shadow: 0 4px 20px rgba(255,107,107,0.35);
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  position: relative;
  overflow: hidden;
  animation: cardSlideIn 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}
.start-btn:active { transform: scale(0.97); }

.rules-hint {
  display: flex; flex-direction: column; gap: 10px;
  animation: cardSlideIn 0.5s cubic-bezier(0.22, 1, 0.36, 1) 0.25s both;
}
.rule-item { font-size: 0.85rem; color: rgba(255,255,255,0.4); }

/* ===== Shimmer 按钮 ===== */
.shimmer-btn {
  position: relative;
  overflow: hidden;
}
.shimmer-btn::after {
  content: '';
  position: absolute;
  top: 0; left: -100%; width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  animation: shimmerSweep 2.5s ease-in-out infinite;
}

@keyframes shimmerSweep {
  0% { left: -100%; }
  50%, 100% { left: 100%; }
}

/* ===== 游戏进行中 ===== */
.game-playing { display: flex; flex-direction: column; }

.game-header {
  margin-bottom: 20px;
  animation: cardSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

.turn-indicator {
  display: flex; align-items: center; justify-content: center; gap: 10px;
  padding: 12px 20px;
  background: linear-gradient(135deg, rgba(255,107,107,0.15), rgba(255,142,83,0.1));
  border: 1.5px solid rgba(255,107,107,0.25);
  border-radius: 16px;
  animation: turnPulse 2s ease-in-out infinite;
}

@keyframes turnPulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(255,107,107,0.2); }
  50% { box-shadow: 0 0 0 6px rgba(255,107,107,0); }
}

.turn-label {
  font-size: 0.85rem; color: rgba(255,255,255,0.5);
}

.turn-player {
  font-size: 1.05rem; font-weight: 700; color: #ff6b6b;
}

.range-display {
  display: flex; align-items: center; justify-content: center;
  gap: 20px; margin-bottom: 16px;
  animation: cardSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.1s both;
}
.range-number {
  width: 80px; padding: 14px 8px;
  background: rgba(255,255,255,0.05);
  border: 1.5px solid rgba(255,255,255,0.1);
  border-radius: 16px; text-align: center;
  font-size: 1.8rem; font-weight: 700; color: #fff;
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}
.range-center { text-align: center; }
.bomb-icon {
  font-size: 2rem;
  animation: bomb-bounce 2s infinite;
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.bomb-icon.danger-zone {
  animation: bomb-bounce 2s infinite, bombDanger 0.5s ease-in-out infinite;
  filter: drop-shadow(0 0 12px rgba(255, 40, 40, 0.5));
}

@keyframes bombDanger {
  0%, 100% { transform: translateY(0) scale(1) rotate(0deg); }
  25% { transform: translateY(-2px) scale(1.05) rotate(-3deg); }
  75% { transform: translateY(-2px) scale(1.05) rotate(3deg); }
}

@keyframes bomb-bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
}
.range-label { font-size: 0.75rem; color: rgba(255,255,255,0.4); margin-top: 4px; }

.range-progress {
  margin-bottom: 20px;
  animation: cardSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.15s both;
}
.progress-bar {
  width: 100%; height: 6px;
  background: rgba(255,255,255,0.08); border-radius: 3px; overflow: hidden;
}
.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #ff6b6b, #ff8e53);
  transition: width 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}
.progress-text {
  text-align: center; font-size: 0.8rem;
  color: rgba(255,255,255,0.4); margin-top: 6px;
}

.guess-section {
  display: flex; gap: 10px; margin-bottom: 16px;
  animation: cardSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.2s both;
}
.guess-section input {
  flex: 1; height: 56px; padding: 0 18px;
  background: rgba(255,255,255,0.05);
  border: 1.5px solid rgba(255,255,255,0.1);
  border-radius: 16px; color: #fff;
  font-size: 1.3rem; text-align: center; font-weight: 600;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.guess-section input:focus {
  outline: none;
  border-color: rgba(255,107,107,0.5);
  box-shadow: 0 0 0 3px rgba(255,107,107,0.12), 0 0 24px rgba(255,107,107,0.06);
}

.guess-btn {
  width: 72px; height: 56px;
  background: linear-gradient(135deg, #ff6b6b, #ff8e53);
  border: none; border-radius: 16px;
  color: #fff; font-size: 0.95rem; font-weight: 600; cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.guess-btn:active:not(:disabled) { transform: scale(0.95); }
.guess-btn:disabled { opacity: 0.4; }

/* 迷你玩家列表 */
.mini-players {
  display: flex; gap: 8px; flex-wrap: wrap;
  justify-content: center; margin-bottom: 16px;
  animation: cardSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.25s both;
}
.mini-player {
  padding: 6px 14px; border-radius: 20px;
  background: rgba(255,255,255,0.05);
  font-size: 0.8rem; color: rgba(255,255,255,0.5);
  border: 1px solid rgba(255,255,255,0.06);
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}
.mini-player.active {
  background: rgba(255,107,107,0.15);
  border-color: rgba(255,107,107,0.3);
  color: #ff6b6b; font-weight: 600;
  box-shadow: 0 0 12px rgba(255,107,107,0.15);
  animation: activePulse 1.5s ease-in-out infinite;
}

@keyframes activePulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(255,107,107,0.25), 0 0 12px rgba(255,107,107,0.15); }
  50% { box-shadow: 0 0 0 5px rgba(255,107,107,0), 0 0 18px rgba(255,107,107,0.08); }
}

/* 历史记录 */
.history-section {
  background: rgba(255,255,255,0.03); border-radius: 14px; padding: 14px;
  animation: cardSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.3s both;
}

.history-list {
  display: flex; flex-direction: column; gap: 6px;
}

.history-row {
  display: flex; align-items: center; justify-content: space-between;
  padding: 8px 12px;
  background: rgba(255,255,255,0.03);
  border-radius: 10px;
  animation: historyRowIn 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes historyRowIn {
  from { opacity: 0; transform: translateX(-8px); }
  to { opacity: 1; transform: translateX(0); }
}

.history-player {
  font-size: 0.82rem; color: rgba(255,255,255,0.5); font-weight: 500;
}

.history-guess {
  font-size: 0.95rem; font-weight: 700; color: rgba(255,255,255,0.8);
  min-width: 40px; text-align: center;
  padding: 2px 10px;
  background: rgba(255,255,255,0.06);
  border-radius: 8px;
}

@keyframes cardSlideIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

/* ===== 爆炸弹窗 ===== */
.explosion-overlay {
  position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.92);
  display: flex; align-items: center; justify-content: center;
  padding: 20px; z-index: 200;
}
.explosion-content {
  text-align: center;
  animation: explosionContentIn 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes explosionContentIn {
  0% { opacity: 0; transform: scale(0.5); }
  50% { transform: scale(1.08); }
  70% { transform: scale(0.97); }
  100% { opacity: 1; transform: scale(1); }
}

.boom-icon {
  font-size: 5rem; margin-bottom: 16px;
  animation: boomPulse 0.8s cubic-bezier(0.22, 1, 0.36, 1);
}
@keyframes boomPulse {
  0% { transform: scale(0); opacity: 0; }
  50% { transform: scale(1.3); opacity: 1; }
  100% { transform: scale(1); opacity: 1; }
}
.boom-title {
  font-size: 2.5rem; color: #ff4757; font-weight: 800; margin-bottom: 10px;
  text-shadow: 0 0 40px rgba(255, 71, 87, 0.4);
  animation: titleGlow 1.5s ease-in-out infinite;
}

@keyframes titleGlow {
  0%, 100% { text-shadow: 0 0 40px rgba(255, 71, 87, 0.4); }
  50% { text-shadow: 0 0 60px rgba(255, 71, 87, 0.6), 0 0 80px rgba(255, 71, 87, 0.2); }
}

.boom-player { font-size: 1.2rem; color: rgba(255,255,255,0.9); margin-bottom: 16px; }
.boom-number { font-size: 1rem; color: rgba(255,255,255,0.6); margin-bottom: 10px; }
.bomb-value {
  font-size: 1.8rem; color: #ff6b6b; font-weight: 700;
  text-shadow: 0 0 20px rgba(255,107,107,0.4);
}
.boom-msg { font-size: 1.1rem; color: rgba(255,255,255,0.8); margin-bottom: 28px; }

.result-actions {
  display: flex; flex-direction: column; gap: 12px; align-items: center;
}

.restart-btn {
  width: 220px; height: 52px;
  background: linear-gradient(135deg, #ff6b6b, #ff8e53);
  border: none; border-radius: 16px;
  color: #fff; font-size: 1.05rem; font-weight: 600;
  cursor: pointer; box-shadow: 0 4px 20px rgba(255,107,107,0.35);
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  position: relative;
  overflow: hidden;
}
.restart-btn:active { transform: scale(0.97); }

.reset-btn {
  width: 220px; height: 48px;
  background: rgba(255,255,255,0.06);
  border: 1.5px solid rgba(255,255,255,0.12);
  border-radius: 16px;
  color: rgba(255,255,255,0.7); font-size: 0.95rem; font-weight: 500;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.reset-btn:active { transform: scale(0.97); background: rgba(255,255,255,0.1); }

.error-toast {
  background: rgba(255,107,107,0.12);
  border: 1px solid rgba(255,107,107,0.25);
  border-radius: 12px; padding: 12px 16px;
  font-size: 0.85rem; color: #ff6b6b;
  text-align: center; margin-bottom: 14px;
  animation: cardSlideIn 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.waiting-text { color: rgba(255,255,255,0.4); font-size: 0.9rem; margin-top: 8px; }

@media (max-height: 700px) {
  .intro-emoji { font-size: 2.8rem; }
  .range-number { width: 65px; padding: 10px 6px; font-size: 1.4rem; }
  .start-btn, .restart-btn { height: 48px; }
}
</style>
