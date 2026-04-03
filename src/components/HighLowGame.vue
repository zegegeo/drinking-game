<script setup>
import { ref, computed } from 'vue'

defineEmits(['goBack'])

// 一副扑克（不含大小王）
const suits = ['♠', '♥', '♦', '♣']
const ranks = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']
const rankValues = { A: 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9, '10': 10, J: 11, Q: 12, K: 13 }

const gameStarted = ref(false)
const deck = ref([])
const currentCard = ref(null)
const nextCard = ref(null)
const isRevealing = ref(false)
const result = ref(null) // 'correct' | 'wrong'
const streak = ref(0)
const bestStreak = ref(0)
const cardsLeft = ref(0)
const totalWrong = ref(0)

const triggerHaptic = () => {
  if ('vibrate' in navigator) navigator.vibrate(25)
}

const createDeck = () => {
  const d = []
  for (const suit of suits) {
    for (const rank of ranks) {
      d.push({ suit, rank, value: rankValues[rank] })
    }
  }
  // Fisher-Yates shuffle
  for (let i = d.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [d[i], d[j]] = [d[j], d[i]]
  }
  return d
}

const startGame = () => {
  triggerHaptic()
  deck.value = createDeck()
  currentCard.value = deck.value.pop()
  nextCard.value = deck.value.pop()
  streak.value = 0
  totalWrong.value = 0
  result.value = null
  isRevealing.value = false
  cardsLeft.value = deck.value.length
  gameStarted.value = true
}

const guess = (direction) => {
  if (isRevealing.value) return
  triggerHaptic()
  isRevealing.value = true

  const curr = currentCard.value.value
  const next = nextCard.value.value

  let correct = false
  if (direction === 'high') {
    correct = next >= curr
  } else {
    correct = next <= curr
  }

  result.value = correct ? 'correct' : 'wrong'

  if (correct) {
    streak.value++
    if (streak.value > bestStreak.value) bestStreak.value = streak.value
  } else {
    totalWrong.value++
    streak.value = 0
    if ('vibrate' in navigator) navigator.vibrate([50, 30, 50])
  }

  setTimeout(() => {
    // 翻到下一张
    currentCard.value = nextCard.value
    if (deck.value.length > 0) {
      nextCard.value = deck.value.pop()
      cardsLeft.value = deck.value.length
    } else {
      // 牌用完了，重新洗牌
      deck.value = createDeck()
      nextCard.value = deck.value.pop()
      cardsLeft.value = deck.value.length
    }
    result.value = null
    isRevealing.value = false
  }, 1200)
}

const isRed = (suit) => suit === '♥' || suit === '♦'

const backToMenu = () => {
  triggerHaptic()
  gameStarted.value = false
}
</script>

<template>
  <div class="hl-mobile">
    <div class="hl-bg">
      <div class="hl-mesh hl-mesh-1"></div>
      <div class="hl-mesh hl-mesh-2"></div>
    </div>
    <div class="hl-grain"></div>

    <header class="mobile-header">
      <button class="back-btn" @click="gameStarted ? backToMenu() : $emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title">猜大小</h1>
      <div class="header-spacer"></div>
    </header>

    <main class="game-area">
      <!-- 开始界面 -->
      <div v-if="!gameStarted" class="setup-screen">
        <div class="setup-icon-wrap">
          <div class="setup-icon-glow"></div>
          <span class="setup-icon">🃏</span>
        </div>
        <h2 class="setup-title">猜大小</h2>
        <p class="setup-desc">翻开一张牌，猜下一张比它大还是小？猜错就喝酒！连对越多越厉害</p>

        <button class="start-btn" @click="startGame">
          <span class="btn-shimmer"></span>
          开始游戏
        </button>
      </div>

      <!-- 游戏界面 -->
      <div v-else class="game-screen">
        <!-- 分数栏 -->
        <div class="score-bar">
          <div class="score-item">
            <span class="score-label">连对</span>
            <span class="score-value" :class="{ hot: streak >= 3 }">{{ streak }}</span>
          </div>
          <div class="score-item">
            <span class="score-label">最佳</span>
            <span class="score-value best">{{ bestStreak }}</span>
          </div>
          <div class="score-item">
            <span class="score-label">喝酒</span>
            <span class="score-value drinks">🍺 {{ totalWrong }}</span>
          </div>
          <div class="score-item">
            <span class="score-label">剩余</span>
            <span class="score-value">{{ cardsLeft }}</span>
          </div>
        </div>

        <!-- 卡片区域 -->
        <div class="cards-area">
          <!-- 当前牌 -->
          <div class="playing-card current-card" :class="{ red: isRed(currentCard.suit) }">
            <div class="pc-corner top-left">
              <div class="pc-rank">{{ currentCard.rank }}</div>
              <div class="pc-suit">{{ currentCard.suit }}</div>
            </div>
            <div class="pc-center">{{ currentCard.suit }}</div>
            <div class="pc-corner bottom-right">
              <div class="pc-rank">{{ currentCard.rank }}</div>
              <div class="pc-suit">{{ currentCard.suit }}</div>
            </div>
          </div>

          <!-- 下一张牌 -->
          <Transition name="card-reveal" mode="out-in">
            <!-- 已揭晓 -->
            <div v-if="isRevealing" key="revealed"
              class="playing-card next-card"
              :class="[
                isRed(nextCard.suit) ? 'red' : '',
                result
              ]"
            >
              <div class="pc-corner top-left">
                <div class="pc-rank">{{ nextCard.rank }}</div>
                <div class="pc-suit">{{ nextCard.suit }}</div>
              </div>
              <div class="pc-center">{{ nextCard.suit }}</div>
              <div class="pc-corner bottom-right">
                <div class="pc-rank">{{ nextCard.rank }}</div>
                <div class="pc-suit">{{ nextCard.suit }}</div>
              </div>
              <div class="result-overlay" :class="result">
                {{ result === 'correct' ? '✓' : '✗' }}
              </div>
            </div>
            <!-- 未揭晓 -->
            <div v-else key="hidden" class="playing-card next-card card-back-face">
              <div class="card-back-pattern"></div>
              <div class="card-back-icon">?</div>
            </div>
          </Transition>
        </div>

        <!-- 结果提示 -->
        <Transition name="fade">
          <div v-if="result" class="result-hint" :class="result">
            <template v-if="result === 'correct'">
              🎉 猜对了！连对 {{ streak }} 次
            </template>
            <template v-else>
              💥 猜错了！请喝酒一杯
            </template>
          </div>
        </Transition>

        <!-- 操作按钮 -->
        <div class="guess-buttons">
          <button class="guess-btn low-btn" @click="guess('low')" :disabled="isRevealing">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <path d="M12 5v14M5 12l7 7 7-7"/>
            </svg>
            <span>更小</span>
          </button>
          <div class="vs-badge">VS</div>
          <button class="guess-btn high-btn" @click="guess('high')" :disabled="isRevealing">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <path d="M12 19V5M5 12l7-7 7 7"/>
            </svg>
            <span>更大</span>
          </button>
        </div>

        <div class="game-tip">相同点数也算猜对</div>
      </div>
    </main>
  </div>
</template>

<style scoped>
.hl-mobile {
  position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: linear-gradient(160deg, #0f0c29 0%, #1a1040 45%, #120e2a 100%);
  display: flex; flex-direction: column; overflow: hidden;
}
.hl-bg { position: fixed; top: 0; left: 0; right: 0; bottom: 0; pointer-events: none; z-index: 0; }
.hl-mesh { position: absolute; top: 0; left: 0; right: 0; bottom: 0; }
.hl-mesh-1 {
  background: radial-gradient(ellipse 55% 45% at 20% 20%, rgba(102,126,234,0.15), transparent),
              radial-gradient(ellipse 45% 40% at 80% 25%, rgba(240,171,251,0.1), transparent);
  animation: hlDrift1 22s ease-in-out infinite alternate;
}
.hl-mesh-2 {
  background: radial-gradient(ellipse 45% 50% at 65% 70%, rgba(79,172,254,0.1), transparent);
  animation: hlDrift2 26s ease-in-out infinite alternate;
}
@keyframes hlDrift1 { 0% { transform: translate(0,0) scale(1); } 100% { transform: translate(12px,-8px) scale(1.03); } }
@keyframes hlDrift2 { 0% { transform: translate(0,0) scale(1); } 100% { transform: translate(-10px,12px) scale(1.05); } }

.hl-grain {
  position: fixed; top: -50%; left: -50%; width: 200%; height: 200%;
  pointer-events: none; z-index: 1; opacity: 0.022;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  background-size: 128px 128px;
}

.mobile-header {
  display: flex; align-items: center; padding: 12px 16px;
  padding-top: max(12px, env(safe-area-inset-top));
  background: rgba(15,12,41,0.6); backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px);
  border-bottom: 1px solid rgba(255,255,255,0.06); z-index: 10; position: relative;
}
.back-btn {
  width: 44px; height: 44px; display: flex; align-items: center; justify-content: center;
  background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.08);
  border-radius: 14px; color: rgba(255,255,255,0.85); cursor: pointer; transition: all 0.15s ease;
}
.back-btn:active { background: rgba(255,255,255,0.12); transform: scale(0.97); }
.header-title {
  flex: 1; text-align: center; font-size: 1.1rem; font-weight: 700; margin: 0 8px;
  background: linear-gradient(135deg, #667eea 0%, #a78bfa 50%, #f093fb 100%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.header-spacer { width: 44px; }

.game-area {
  flex: 1; overflow-y: auto; padding: 20px 16px;
  padding-bottom: max(24px, env(safe-area-inset-bottom));
  display: flex; flex-direction: column; position: relative; z-index: 2;
}

/* Setup */
.setup-screen { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; max-width: 360px; margin: 0 auto; width: 100%; }
.setup-icon-wrap { position: relative; width: 80px; height: 80px; margin-bottom: 16px; display: flex; align-items: center; justify-content: center; }
.setup-icon { font-size: 3rem; position: relative; z-index: 2; animation: iconFloat 3.5s ease-in-out infinite; }
@keyframes iconFloat { 0%,100% { transform: translateY(0); } 50% { transform: translateY(-6px); } }
.setup-icon-glow {
  position: absolute; top: 50%; left: 50%; width: 70px; height: 70px; transform: translate(-50%,-50%);
  background: radial-gradient(circle, rgba(102,126,234,0.3), transparent 70%);
  border-radius: 50%; filter: blur(14px); z-index: 0;
}
.setup-title { font-size: 1.5rem; font-weight: 800; color: rgba(255,255,255,0.92); margin: 0 0 8px; }
.setup-desc { font-size: 0.85rem; color: rgba(255,255,255,0.35); text-align: center; line-height: 1.6; margin: 0 0 36px; max-width: 280px; }
.start-btn {
  width: 100%; max-width: 300px; height: 52px; border-radius: 18px; border: none; font-size: 1rem; font-weight: 600;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: #fff;
  cursor: pointer; position: relative; overflow: hidden; transition: all 0.15s ease;
  box-shadow: 0 4px 20px rgba(102,126,234,0.3);
}
.start-btn:active { transform: scale(0.97); }
.btn-shimmer {
  position: absolute; top: 0; left: -100%; width: 60%; height: 100%;
  background: linear-gradient(105deg, transparent 35%, rgba(255,255,255,0.25) 50%, transparent 65%);
  animation: shimmerSweep 2.8s cubic-bezier(0.22,1,0.36,1) infinite; pointer-events: none;
}
@keyframes shimmerSweep { 0% { left: -100%; } 40% { left: 160%; } 100% { left: 160%; } }

/* Game Screen */
.game-screen {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  max-width: 400px; margin: 0 auto; width: 100%;
}

.score-bar { display: flex; gap: 8px; width: 100%; margin-bottom: 20px; }
.score-item {
  flex: 1; text-align: center; padding: 10px 6px;
  background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.06);
  border-radius: 14px;
}
.score-label { display: block; font-size: 0.68rem; color: rgba(255,255,255,0.35); margin-bottom: 2px; }
.score-value { display: block; font-size: 1.15rem; font-weight: 700; color: rgba(255,255,255,0.9); }
.score-value.hot { color: #ff6b6b; animation: hotPulse 0.6s ease-in-out infinite; }
@keyframes hotPulse { 0%,100% { transform: scale(1); } 50% { transform: scale(1.1); } }
.score-value.best { color: #ffd700; }
.score-value.drinks { font-size: 0.95rem; }

/* Cards */
.cards-area {
  display: flex; gap: 16px; align-items: center; justify-content: center;
  margin-bottom: 16px;
}

.playing-card {
  width: 120px; height: 170px; border-radius: 14px; position: relative;
  display: flex; align-items: center; justify-content: center;
  box-shadow: 0 8px 28px rgba(0,0,0,0.4);
  transition: transform 0.15s ease;
}

.playing-card:not(.card-back-face) {
  background: rgba(255,255,255,0.95);
  border: 1px solid rgba(255,255,255,0.2);
  color: #1a1a2e;
}

.playing-card.red { color: #dc3545; }

.pc-corner {
  position: absolute; display: flex; flex-direction: column; align-items: center; line-height: 1;
}
.top-left { top: 8px; left: 10px; }
.bottom-right { bottom: 8px; right: 10px; transform: rotate(180deg); }
.pc-rank { font-size: 1rem; font-weight: 800; }
.pc-suit { font-size: 0.75rem; }
.pc-center { font-size: 2.8rem; opacity: 0.15; }

/* Card back */
.card-back-face {
  background: linear-gradient(135deg, #667eea, #764ba2);
  border: 2px solid rgba(255,255,255,0.15);
}
.card-back-pattern {
  position: absolute; inset: 6px; border-radius: 8px;
  border: 1.5px solid rgba(255,255,255,0.15);
  background-image: repeating-linear-gradient(45deg, transparent, transparent 8px, rgba(255,255,255,0.04) 8px, rgba(255,255,255,0.04) 16px);
}
.card-back-icon {
  font-size: 2rem; font-weight: 800; color: rgba(255,255,255,0.4);
  position: relative; z-index: 1;
}

/* Result overlay on card */
.result-overlay {
  position: absolute; top: 0; left: 0; right: 0; bottom: 0;
  border-radius: 14px; display: flex; align-items: center; justify-content: center;
  font-size: 3rem; font-weight: 800;
  animation: resultPop 0.3s cubic-bezier(0.34,1.56,0.64,1);
}
.result-overlay.correct { background: rgba(67,233,123,0.2); color: #43e97b; }
.result-overlay.wrong { background: rgba(255,107,107,0.2); color: #ff6b6b; }
@keyframes resultPop { from { transform: scale(0); } to { transform: scale(1); } }

.next-card.correct { box-shadow: 0 0 24px rgba(67,233,123,0.3), 0 8px 28px rgba(0,0,0,0.3); }
.next-card.wrong { box-shadow: 0 0 24px rgba(255,107,107,0.3), 0 8px 28px rgba(0,0,0,0.3); animation: cardShake 0.4s ease; }
@keyframes cardShake {
  0%,100% { transform: translateX(0); }
  20% { transform: translateX(-6px); }
  40% { transform: translateX(6px); }
  60% { transform: translateX(-4px); }
  80% { transform: translateX(4px); }
}

/* Card reveal transition */
.card-reveal-enter-active { animation: cardFlipIn 0.35s cubic-bezier(0.22,1,0.36,1); }
.card-reveal-leave-active { animation: cardFlipOut 0.2s ease-out; }
@keyframes cardFlipIn { from { opacity: 0; transform: scale(0.85) rotateY(10deg); } to { opacity: 1; transform: scale(1) rotateY(0); } }
@keyframes cardFlipOut { from { opacity: 1; transform: scale(1); } to { opacity: 0; transform: scale(0.9); } }

/* Result hint */
.result-hint {
  text-align: center; font-size: 0.95rem; font-weight: 600; padding: 10px 20px;
  border-radius: 14px; margin-bottom: 16px; min-height: 42px;
}
.result-hint.correct { background: rgba(67,233,123,0.1); color: #43e97b; border: 1px solid rgba(67,233,123,0.15); }
.result-hint.wrong { background: rgba(255,107,107,0.1); color: #ff6b6b; border: 1px solid rgba(255,107,107,0.15); }

.fade-enter-active { animation: fadeIn 0.3s ease; }
.fade-leave-active { animation: fadeOut 0.2s ease forwards; }
@keyframes fadeIn { from { opacity: 0; transform: translateY(6px); } to { opacity: 1; transform: translateY(0); } }
@keyframes fadeOut { to { opacity: 0; } }

/* Guess buttons */
.guess-buttons {
  display: flex; gap: 12px; align-items: center; width: 100%;
  margin-bottom: 12px;
}
.guess-btn {
  flex: 1; height: 64px; border-radius: 20px; border: none;
  display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 2px;
  font-size: 1rem; font-weight: 700; cursor: pointer; transition: all 0.15s ease;
  touch-action: manipulation;
}
.guess-btn:active:not(:disabled) { transform: scale(0.96); }
.guess-btn:disabled { opacity: 0.35; cursor: not-allowed; }
.guess-btn span { font-size: 0.85rem; }

.low-btn {
  background: rgba(79,172,254,0.1); border: 1.5px solid rgba(79,172,254,0.25) !important;
  color: #4facfe;
}
.low-btn:active:not(:disabled) { box-shadow: 0 0 24px rgba(79,172,254,0.2); }

.high-btn {
  background: rgba(255,107,107,0.1); border: 1.5px solid rgba(255,107,107,0.25) !important;
  color: #ff6b6b;
}
.high-btn:active:not(:disabled) { box-shadow: 0 0 24px rgba(255,107,107,0.2); }

.vs-badge {
  width: 36px; height: 36px; border-radius: 50%; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255,255,255,0.06); border: 1px solid rgba(255,255,255,0.1);
  font-size: 0.72rem; font-weight: 800; color: rgba(255,255,255,0.3);
  letter-spacing: 1px;
}

.game-tip {
  font-size: 0.72rem; color: rgba(255,255,255,0.2); text-align: center;
}

@media (max-height: 700px) {
  .playing-card { width: 100px; height: 145px; }
  .pc-center { font-size: 2.2rem; }
  .guess-btn { height: 56px; }
  .score-bar { margin-bottom: 12px; }
}

@media (max-height: 600px) {
  .playing-card { width: 85px; height: 125px; }
  .cards-area { margin-bottom: 10px; }
  .guess-btn { height: 50px; }
}
</style>
