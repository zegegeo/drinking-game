<script setup>
import { ref, computed } from 'vue'

defineEmits(['goBack'])

// ==================== 游戏状态 ====================
const gameStatus = ref('setup') // 'setup' | 'playing' | 'gameover'
const playerCount = ref(2)
const players = ref([])
const currentPlayerIndex = ref(0)
const deck = ref([])

const suits = ['♠', '♥', '♦', '♣']
const values = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']

// ==================== 工具函数 ====================
const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(25)
  }
}

const createDeck = () => {
  deck.value = []
  // 用2副牌，确保多人游戏有足够的牌
  for (let d = 0; d < 2; d++) {
    for (let suit of suits) {
      for (let value of values) {
        deck.value.push({ suit, value })
      }
    }
  }
  shuffleDeck()
}

const shuffleDeck = () => {
  for (let i = deck.value.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[deck.value[i], deck.value[j]] = [deck.value[j], deck.value[i]]
  }
}

const drawCard = () => {
  return deck.value.pop() || { suit: '♠', value: 'A' }
}

const getCardValue = (card) => {
  if (['J', 'Q', 'K'].includes(card.value)) return 10
  if (card.value === 'A') return 11
  return parseInt(card.value)
}

const calculateScore = (hand) => {
  let score = 0
  let aces = 0
  for (let card of hand) {
    score += getCardValue(card)
    if (card.value === 'A') aces++
  }
  while (score > 21 && aces > 0) {
    score -= 10
    aces--
  }
  return score
}

const getCardStyle = (card) => {
  const isRed = ['♥', '♦'].includes(card.suit)
  return { color: isRed ? '#ef4444' : '#1e293b' }
}

// ==================== 当前玩家 ====================
const currentPlayer = computed(() => {
  if (players.value.length === 0) return null
  return players.value[currentPlayerIndex.value] || null
})

// 存活（未爆牌、未停牌）的玩家
const allDone = computed(() => {
  return players.value.every(p => p.busted || p.stood)
})

// 爆牌的玩家列表
const bustedPlayers = computed(() => {
  return players.value.filter(p => p.busted)
})

// 没爆牌的玩家列表
const survivedPlayers = computed(() => {
  return players.value.filter(p => !p.busted)
})

// ==================== 设置阶段 ====================
const setPlayerCount = (count) => {
  triggerHaptic()
  playerCount.value = count
}

const startGame = () => {
  triggerHaptic()
  createDeck()

  players.value = []
  for (let i = 0; i < playerCount.value; i++) {
    players.value.push({
      name: `玩家${i + 1}`,
      hand: [drawCard(), drawCard()],
      score: 0,
      busted: false,
      stood: false
    })
  }
  // 计算初始分数
  for (let p of players.value) {
    p.score = calculateScore(p.hand)
  }

  // 找到第一个可以操作的玩家
  currentPlayerIndex.value = 0
  gameStatus.value = 'playing'
}

// ==================== 游戏操作 ====================
const advanceToNextPlayer = () => {
  // 找下一个还没爆牌也没停牌的玩家
  let next = currentPlayerIndex.value + 1
  while (next < players.value.length) {
    if (!players.value[next].busted && !players.value[next].stood) {
      currentPlayerIndex.value = next
      return
    }
    next++
  }
  // 所有人都完成了
  endGame()
}

const hit = () => {
  triggerHaptic()
  const p = players.value[currentPlayerIndex.value]
  if (!p || p.busted || p.stood) return

  p.hand.push(drawCard())
  p.score = calculateScore(p.hand)

  if (p.score > 21) {
    p.busted = true
    // 检查是否所有人完成
    if (allDone.value) {
      endGame()
    } else {
      advanceToNextPlayer()
    }
  } else if (p.score === 21) {
    // 21点自动停牌
    p.stood = true
    if (allDone.value) {
      endGame()
    } else {
      advanceToNextPlayer()
    }
  }
}

const stand = () => {
  triggerHaptic()
  const p = players.value[currentPlayerIndex.value]
  if (!p || p.busted || p.stood) return

  p.stood = true

  if (allDone.value) {
    endGame()
  } else {
    advanceToNextPlayer()
  }
}

const endGame = () => {
  gameStatus.value = 'gameover'
}

// ==================== 重新开始 ====================
const playAgain = () => {
  triggerHaptic()
  startGame()
}

const backToSetup = () => {
  triggerHaptic()
  gameStatus.value = 'setup'
  players.value = []
  currentPlayerIndex.value = 0
}
</script>

<template>
  <div class="blackjack-mobile">
    <!-- 顶部标题栏 -->
    <header class="mobile-header">
      <button class="back-btn" @click="$emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title bj-title">21点</h1>
      <div class="header-spacer"></div>
    </header>

    <!-- 主游戏区域 -->
    <main class="game-area">
      <!-- ========== 设置阶段 ========== -->
      <div v-if="gameStatus === 'setup'" class="setup-phase">
        <div class="setup-intro">
          <div class="intro-icon">🃏</div>
          <div class="intro-text">多人21点</div>
          <div class="intro-sub">选择玩家数量开始游戏</div>
        </div>

        <div class="player-count-selector">
          <div class="selector-label">玩家数量</div>
          <div class="count-options">
            <button
              v-for="(n, idx) in [2, 3, 4, 5, 6]"
              :key="n"
              class="count-btn stagger-in"
              :class="{ active: playerCount === n }"
              :style="{ animationDelay: idx * 0.07 + 's' }"
              @click="setPlayerCount(n)"
            >
              {{ n }}人
            </button>
          </div>
        </div>

        <button class="start-btn shimmer-btn" @click="startGame">
          开始游戏
        </button>
      </div>

      <!-- ========== 游戏进行中 ========== -->
      <div v-else-if="gameStatus === 'playing'" class="playing-phase">
        <!-- 当前操作提示 -->
        <div class="turn-indicator">
          <span class="turn-text">当前操作：</span>
          <span class="turn-player-name">{{ currentPlayer?.name }}</span>
        </div>

        <!-- 所有玩家区域 -->
        <div class="players-list">
          <div
            v-for="(player, index) in players"
            :key="index"
            class="player-section"
            :class="{
              'player-active': index === currentPlayerIndex && !player.busted && !player.stood,
              'player-busted': player.busted,
              'player-stood': player.stood && !player.busted
            }"
          >
            <div class="section-header">
              <span class="player-name">
                {{ player.name }}
                <span v-if="index === currentPlayerIndex && !player.busted && !player.stood" class="active-tag">操作中</span>
                <span v-else-if="player.busted" class="bust-tag">爆牌!</span>
                <span v-else-if="player.stood" class="stood-tag">停牌</span>
              </span>
              <span
                class="score-badge"
                :class="{
                  'score-bust': player.busted,
                  'score-blackjack': player.score === 21
                }"
              >
                {{ player.score }}
              </span>
            </div>
            <div class="cards-row">
              <div
                v-for="(card, ci) in player.hand"
                :key="ci"
                class="card card-deal-in"
                :style="[getCardStyle(card), { animationDelay: ci * 0.15 + 's' }]"
              >
                <div class="card-front">
                  <span class="card-corner card-corner-top">
                    <span class="corner-value">{{ card.value }}</span>
                    <span class="corner-suit">{{ card.suit }}</span>
                  </span>
                  <div class="card-center-suit">{{ card.suit }}</div>
                  <span class="card-corner card-corner-bottom">
                    <span class="corner-value">{{ card.value }}</span>
                    <span class="corner-suit">{{ card.suit }}</span>
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 操作按钮 -->
        <div class="action-buttons">
          <button class="hit-btn shimmer-btn" @click="hit">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <path d="M12 5v14M5 12h14"/>
            </svg>
            要牌
          </button>
          <button class="stand-btn shimmer-btn" @click="stand">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <path d="M5 12h14"/>
            </svg>
            停牌
          </button>
        </div>
      </div>

      <!-- ========== 游戏结束 ========== -->
      <div v-else class="gameover-phase">
        <div class="result-title">游戏结束</div>

        <!-- 爆牌的玩家 -->
        <div v-if="bustedPlayers.length > 0" class="result-section">
          <div class="result-section-title bust-title">
            <span class="result-icon">💥</span> 爆牌喝酒
          </div>
          <div class="result-players bust-players">
            <div v-for="(p, i) in bustedPlayers" :key="'bust-' + i" class="result-player-card busted-card">
              <div class="result-player-name">{{ p.name }}</div>
              <div class="result-player-score">{{ p.score }}点</div>
              <div class="result-player-drink drink-pulse">🍺 喝酒！</div>
            </div>
          </div>
        </div>

        <!-- 没爆牌的玩家 -->
        <div v-if="survivedPlayers.length > 0" class="result-section">
          <div class="result-section-title safe-title">
            <span class="result-icon">✨</span> 存活玩家
          </div>
          <div class="result-players safe-players">
            <div v-for="(p, i) in survivedPlayers" :key="'safe-' + i" class="result-player-card safe-card">
              <div class="result-player-name">{{ p.name }}</div>
              <div class="result-player-score">{{ p.score }}点</div>
              <div class="cards-row cards-row-small">
                <div
                  v-for="(card, ci) in p.hand"
                  :key="ci"
                  class="card card-small card-deal-in"
                  :style="[getCardStyle(card), { animationDelay: ci * 0.1 + 's' }]"
                >
                  <div class="card-front">
                    <span class="card-corner card-corner-top">
                      <span class="corner-value">{{ card.value }}</span>
                      <span class="corner-suit">{{ card.suit }}</span>
                    </span>
                    <div class="card-center-suit">{{ card.suit }}</div>
                    <span class="card-corner card-corner-bottom">
                      <span class="corner-value">{{ card.value }}</span>
                      <span class="corner-suit">{{ card.suit }}</span>
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="compare-hint">
            没爆牌的玩家请自行比较点数，最小的人喝酒！🍻
          </div>
        </div>

        <!-- 全员爆牌 -->
        <div v-if="survivedPlayers.length === 0" class="all-bust-hint">
          全员爆牌！大家一起喝！🍻
        </div>

        <!-- 操作按钮 -->
        <div class="gameover-buttons">
          <button class="new-game-btn shimmer-btn" @click="playAgain">
            再来一局
          </button>
          <button class="back-setup-btn" @click="backToSetup">
            换人玩
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>
.blackjack-mobile {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(180deg, #0a1a12 0%, #152a1e 50%, #0a1a12 100%);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding-bottom: env(safe-area-inset-bottom);
}

/* ========== Header ========== */
.mobile-header {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  padding-top: max(12px, env(safe-area-inset-top));
  background: rgba(0, 0, 0, 0.35);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
}

.back-btn {
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.9);
  cursor: pointer;
  transition: all 0.2s;
}

.back-btn:active {
  background: rgba(255, 255, 255, 0.16);
  transform: scale(0.93);
}

.header-title {
  flex: 1;
  text-align: center;
  font-size: 1.15rem;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.03em;
}

/* Deep green gradient title */
.bj-title {
  background: linear-gradient(135deg, #34d399, #059669, #34d399);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-size: 1.2rem;
}

.header-spacer {
  width: 44px;
}

/* ========== Game Area ========== */
.game-area {
  flex: 1;
  overflow-y: auto;
  padding: 20px 16px;
  padding-bottom: max(40px, env(safe-area-inset-bottom));
  display: flex;
  flex-direction: column;
}

/* ========== Setup Phase ========== */
.setup-phase {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.setup-intro {
  text-align: center;
  margin-bottom: 40px;
}

.intro-icon {
  font-size: 4rem;
  margin-bottom: 16px;
  filter: drop-shadow(0 4px 12px rgba(0,0,0,0.4));
}

.intro-text {
  font-size: 1.4rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.92);
  letter-spacing: 0.02em;
}

.intro-sub {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 8px;
}

.player-count-selector {
  width: 100%;
  max-width: 360px;
  margin-bottom: 36px;
}

.selector-label {
  font-size: 0.9rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.6);
  text-align: center;
  margin-bottom: 14px;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.count-options {
  display: flex;
  gap: 10px;
  justify-content: center;
}

/* Staggered entrance animation */
@keyframes staggerFadeUp {
  0% {
    opacity: 0;
    transform: translateY(18px) scale(0.9);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.stagger-in {
  animation: staggerFadeUp 0.45s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.count-btn {
  width: 56px;
  height: 56px;
  border: 2px solid rgba(255, 255, 255, 0.12);
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.05);
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  display: flex;
  align-items: center;
  justify-content: center;
}

.count-btn:active {
  transform: scale(0.93);
}

.count-btn.active {
  background: rgba(52, 211, 153, 0.15);
  border-color: rgba(52, 211, 153, 0.5);
  color: #34d399;
  box-shadow: 0 0 20px rgba(52, 211, 153, 0.15);
}

/* ========== Shimmer Button Effect ========== */
.shimmer-btn {
  position: relative;
  overflow: hidden;
}

.shimmer-btn::after {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 60%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent 0%,
    rgba(255, 255, 255, 0.15) 40%,
    rgba(255, 255, 255, 0.25) 50%,
    rgba(255, 255, 255, 0.15) 60%,
    transparent 100%
  );
  transform: skewX(-20deg);
  animation: shimmerSweep 3s cubic-bezier(0.22, 1, 0.36, 1) infinite;
  pointer-events: none;
}

@keyframes shimmerSweep {
  0% { left: -100%; }
  40%, 100% { left: 150%; }
}

.start-btn {
  width: 100%;
  max-width: 360px;
  padding: 18px;
  border: none;
  border-radius: 18px;
  background: linear-gradient(135deg, #34d399 0%, #059669 100%);
  color: #fff;
  font-size: 1.1rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 24px rgba(52, 211, 153, 0.3), 0 2px 8px rgba(0,0,0,0.2);
  transition: all 0.2s cubic-bezier(0.22, 1, 0.36, 1);
  letter-spacing: 0.05em;
}

.start-btn:active {
  transform: scale(0.96);
  box-shadow: 0 2px 12px rgba(52, 211, 153, 0.3);
}

/* ========== Playing Phase ========== */
.playing-phase {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.turn-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 10px 18px;
  background: rgba(52, 211, 153, 0.1);
  border: 1px solid rgba(52, 211, 153, 0.2);
  border-radius: 14px;
  flex-shrink: 0;
}

.turn-text {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.6);
}

.turn-player-name {
  font-size: 1rem;
  font-weight: 700;
  color: #34d399;
  text-shadow: 0 0 10px rgba(52, 211, 153, 0.3);
}

.players-list {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 10px;
  overflow-y: auto;
  padding-bottom: 4px;
}

.player-section {
  background: rgba(255, 255, 255, 0.04);
  border: 2px solid rgba(255, 255, 255, 0.06);
  border-radius: 18px;
  padding: 14px;
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

/* Breathing pulse border for active player */
.player-active {
  border-color: rgba(52, 211, 153, 0.5);
  background: rgba(52, 211, 153, 0.08);
  box-shadow: 0 0 24px rgba(52, 211, 153, 0.1);
  animation: breathingPulse 2s cubic-bezier(0.22, 1, 0.36, 1) infinite;
}

@keyframes breathingPulse {
  0%, 100% {
    border-color: rgba(52, 211, 153, 0.5);
    box-shadow: 0 0 16px rgba(52, 211, 153, 0.08), inset 0 0 0 0 rgba(52, 211, 153, 0);
  }
  50% {
    border-color: rgba(52, 211, 153, 0.8);
    box-shadow: 0 0 28px rgba(52, 211, 153, 0.2), inset 0 0 12px rgba(52, 211, 153, 0.04);
  }
}

/* Bust flash red + shake */
.player-busted {
  border-color: rgba(239, 68, 68, 0.3);
  background: rgba(239, 68, 68, 0.06);
  opacity: 0.7;
  animation: bustShake 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes bustShake {
  0% { transform: translateX(0); background: rgba(239, 68, 68, 0.25); }
  10% { transform: translateX(-6px); }
  20% { transform: translateX(6px); background: rgba(239, 68, 68, 0.15); }
  30% { transform: translateX(-5px); }
  40% { transform: translateX(5px); }
  50% { transform: translateX(-3px); background: rgba(239, 68, 68, 0.1); }
  60% { transform: translateX(3px); }
  70% { transform: translateX(-2px); }
  80% { transform: translateX(2px); }
  100% { transform: translateX(0); background: rgba(239, 68, 68, 0.06); }
}

.player-stood {
  border-color: rgba(255, 215, 0, 0.2);
  background: rgba(255, 215, 0, 0.04);
  opacity: 0.8;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.player-name {
  font-size: 0.95rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.7);
  letter-spacing: 0.05em;
  display: flex;
  align-items: center;
  gap: 8px;
}

.active-tag {
  font-size: 0.7rem;
  padding: 2px 8px;
  background: rgba(52, 211, 153, 0.2);
  border: 1px solid rgba(52, 211, 153, 0.3);
  border-radius: 8px;
  color: #34d399;
  font-weight: 600;
  animation: tagPulse 1.5s ease-in-out infinite;
}

@keyframes tagPulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.bust-tag {
  font-size: 0.7rem;
  padding: 2px 8px;
  background: rgba(239, 68, 68, 0.2);
  border: 1px solid rgba(239, 68, 68, 0.3);
  border-radius: 8px;
  color: #ef4444;
  font-weight: 600;
}

.stood-tag {
  font-size: 0.7rem;
  padding: 2px 8px;
  background: rgba(255, 215, 0, 0.15);
  border: 1px solid rgba(255, 215, 0, 0.25);
  border-radius: 8px;
  color: #ffd700;
  font-weight: 600;
}

.score-badge {
  padding: 6px 16px;
  background: rgba(255, 215, 0, 0.12);
  border: 1px solid rgba(255, 215, 0, 0.2);
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 700;
  color: #ffd700;
  min-width: 48px;
  text-align: center;
  transition: all 0.3s;
}

.score-bust {
  background: rgba(239, 68, 68, 0.15);
  border-color: rgba(239, 68, 68, 0.3);
  color: #ef4444;
}

.score-blackjack {
  background: rgba(52, 211, 153, 0.15);
  border-color: rgba(52, 211, 153, 0.3);
  color: #34d399;
  text-shadow: 0 0 10px rgba(52, 211, 153, 0.4);
}

.cards-row {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  min-height: 80px;
}

/* ========== Card Design with dynamic shadow ========== */
@keyframes dealIn {
  0% {
    opacity: 0;
    transform: translateX(80px) rotate(8deg) translateY(-20px);
    box-shadow:
      0 20px 40px rgba(0, 0, 0, 0.5),
      0 12px 24px rgba(0, 0, 0, 0.3);
  }
  50% {
    opacity: 1;
    box-shadow:
      0 14px 30px rgba(0, 0, 0, 0.4),
      0 8px 18px rgba(0, 0, 0, 0.25);
  }
  100% {
    opacity: 1;
    transform: translateX(0) rotate(0deg) translateY(0);
    box-shadow:
      0 4px 12px rgba(0, 0, 0, 0.25),
      0 8px 24px rgba(0, 0, 0, 0.15),
      0 1px 3px rgba(0, 0, 0, 0.2);
  }
}

.card-deal-in {
  animation: dealIn 0.5s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.card {
  width: 52px;
  height: 74px;
  background: #ffffff;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  box-shadow:
    0 4px 12px rgba(0, 0, 0, 0.25),
    0 8px 24px rgba(0, 0, 0, 0.15),
    0 1px 3px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  flex-shrink: 0;
  transition: box-shadow 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.card-front {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  background: linear-gradient(180deg, #ffffff 0%, #f8f8f8 100%);
  border-radius: 7px;
}

.card-corner {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  line-height: 1;
}

.card-corner-top {
  top: 4px;
  left: 5px;
}

.card-corner-bottom {
  bottom: 4px;
  right: 5px;
  transform: rotate(180deg);
}

.corner-value {
  font-size: 0.65rem;
  font-weight: 800;
}

.corner-suit {
  font-size: 0.55rem;
  margin-top: -1px;
}

.card-center-suit {
  font-size: 1.5rem;
}

/* ========== Action Buttons ========== */
.action-buttons {
  display: flex;
  gap: 12px;
  flex-shrink: 0;
  padding-top: 4px;
}

.hit-btn,
.stand-btn {
  flex: 1;
  padding: 16px;
  border: none;
  border-radius: 16px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.2s cubic-bezier(0.22, 1, 0.36, 1);
  letter-spacing: 0.02em;
}

.hit-btn {
  background: linear-gradient(135deg, #34d399 0%, #059669 100%);
  color: #fff;
  box-shadow: 0 4px 18px rgba(52, 211, 153, 0.3), 0 2px 6px rgba(0,0,0,0.2);
}

.hit-btn:active {
  transform: scale(0.96);
  box-shadow: 0 2px 8px rgba(52, 211, 153, 0.3);
}

.stand-btn {
  background: linear-gradient(135deg, #f87171 0%, #dc2626 100%);
  color: #fff;
  box-shadow: 0 4px 18px rgba(239, 68, 68, 0.3), 0 2px 6px rgba(0,0,0,0.2);
}

.stand-btn:active {
  transform: scale(0.96);
  box-shadow: 0 2px 8px rgba(239, 68, 68, 0.3);
}

/* ========== Gameover Phase ========== */
.gameover-phase {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.result-title {
  text-align: center;
  font-size: 1.4rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.92);
  padding: 8px 0;
  animation: resultPop 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}

@keyframes resultPop {
  0% { opacity: 0; transform: scale(0.6); }
  100% { opacity: 1; transform: scale(1); }
}

.result-section {
  margin-bottom: 4px;
}

.result-section-title {
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.bust-title {
  color: #ef4444;
}

.safe-title {
  color: #34d399;
}

.result-icon {
  font-size: 1.1rem;
}

.result-players {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.result-player-card {
  padding: 14px;
  border-radius: 16px;
  flex: 1;
  min-width: 120px;
  text-align: center;
}

.busted-card {
  background: rgba(239, 68, 68, 0.1);
  border: 1px solid rgba(239, 68, 68, 0.25);
}

.safe-card {
  background: rgba(52, 211, 153, 0.08);
  border: 1px solid rgba(52, 211, 153, 0.2);
  min-width: 140px;
}

.result-player-name {
  font-size: 1rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 4px;
}

.result-player-score {
  font-size: 0.9rem;
  font-weight: 600;
  color: #ffd700;
  margin-bottom: 6px;
}

.result-player-drink {
  font-size: 0.9rem;
  font-weight: 600;
  color: #ef4444;
}

/* Drink pulse animation */
.drink-pulse {
  animation: drinkPulseAnim 1.2s cubic-bezier(0.22, 1, 0.36, 1) infinite;
  display: inline-block;
}

@keyframes drinkPulseAnim {
  0%, 100% {
    transform: scale(1);
    text-shadow: 0 0 0 transparent;
  }
  50% {
    transform: scale(1.12);
    text-shadow: 0 0 10px rgba(239, 68, 68, 0.5);
  }
}

.cards-row-small {
  min-height: auto;
  justify-content: center;
  margin-top: 8px;
}

.card-small {
  width: 38px;
  height: 54px;
}

.card-small .card-center-suit {
  font-size: 1rem;
}

.card-small .corner-value {
  font-size: 0.5rem;
}

.card-small .corner-suit {
  font-size: 0.45rem;
}

.card-small .card-corner-top {
  top: 2px;
  left: 3px;
}

.card-small .card-corner-bottom {
  bottom: 2px;
  right: 3px;
}

.compare-hint {
  margin-top: 12px;
  padding: 14px 18px;
  background: rgba(255, 215, 0, 0.08);
  border: 1px solid rgba(255, 215, 0, 0.18);
  border-radius: 14px;
  font-size: 0.95rem;
  font-weight: 600;
  color: #ffd700;
  text-align: center;
  line-height: 1.5;
}

.all-bust-hint {
  padding: 20px;
  background: rgba(239, 68, 68, 0.1);
  border: 1px solid rgba(239, 68, 68, 0.25);
  border-radius: 16px;
  font-size: 1.1rem;
  font-weight: 700;
  color: #ef4444;
  text-align: center;
}

/* ========== Gameover Buttons ========== */
.gameover-buttons {
  display: flex;
  gap: 12px;
  margin-top: auto;
  padding-top: 12px;
}

.new-game-btn,
.back-setup-btn {
  flex: 1;
  padding: 16px;
  border: none;
  border-radius: 16px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.22, 1, 0.36, 1);
  letter-spacing: 0.02em;
}

.new-game-btn {
  background: linear-gradient(135deg, #ffd700 0%, #f0a500 100%);
  color: #1a2e1a;
  box-shadow: 0 4px 24px rgba(255, 215, 0, 0.25), 0 2px 8px rgba(0,0,0,0.2);
  font-weight: 700;
}

.new-game-btn:active {
  transform: scale(0.96);
  box-shadow: 0 2px 12px rgba(255, 215, 0, 0.3);
}

.back-setup-btn {
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.12);
  color: rgba(255, 255, 255, 0.8);
}

.back-setup-btn:active {
  transform: scale(0.96);
  background: rgba(255, 255, 255, 0.14);
}

/* ========== Small Screen ========== */
@media (max-height: 700px) {
  .card {
    width: 44px;
    height: 62px;
  }

  .card-center-suit {
    font-size: 1.2rem;
  }

  .corner-value {
    font-size: 0.55rem;
  }

  .corner-suit {
    font-size: 0.45rem;
  }

  .player-section {
    padding: 10px;
  }

  .cards-row {
    min-height: 62px;
    gap: 6px;
  }

  .section-header {
    margin-bottom: 6px;
  }

  .hit-btn,
  .stand-btn,
  .new-game-btn,
  .back-setup-btn {
    padding: 14px;
  }
}

@media (max-height: 600px) {
  .game-area {
    padding: 12px 12px;
  }

  .player-section {
    padding: 8px;
  }

  .section-header {
    margin-bottom: 4px;
  }

  .turn-indicator {
    padding: 6px 12px;
  }

  .players-list {
    gap: 6px;
  }
}
</style>
