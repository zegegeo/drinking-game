<script setup>
import { ref, computed } from 'vue'

defineEmits(['goBack'])

// 游戏状态
const isDrawing = ref(false)
const showCard = ref(false)
const isFlipping = ref(false)
const currentCard = ref({ suit: '', value: '' })
const drawCount = ref(0)
const kingCount = ref(0) // K的计数，第4个K要喝公杯
const showRules = ref(false)

const suits = ['♠', '♥', '♦', '♣']
const values = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']

// 完整牌堆
const deck = ref([])
const usedCards = ref([])

const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(25)
  }
}

// 初始化牌堆
const initDeck = () => {
  const cards = []
  for (const suit of suits) {
    for (const value of values) {
      cards.push({ suit, value })
    }
  }
  // 洗牌
  for (let i = cards.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[cards[i], cards[j]] = [cards[j], cards[i]]
  }
  deck.value = cards
}

// 剩余牌数
const remainingCards = computed(() => deck.value.length)

// 抽牌
const drawCard = () => {
  if (isDrawing.value) return
  if (deck.value.length === 0) {
    initDeck()
    kingCount.value = 0
    usedCards.value = []
  }

  triggerHaptic()
  isDrawing.value = true
  showCard.value = false
  isFlipping.value = true

  setTimeout(() => {
    const card = deck.value.pop()
    currentCard.value = card
    usedCards.value.push(card)
    drawCount.value++

    if (card.value === 'K') {
      kingCount.value++
    }

    setTimeout(() => {
      showCard.value = true
      isDrawing.value = false
      triggerHaptic()
      setTimeout(() => {
        isFlipping.value = false
      }, 600)
    }, 500)
  }, 400)
}

// 重置游戏
const reset = () => {
  triggerHaptic()
  showCard.value = false
  isFlipping.value = false
  currentCard.value = { suit: '', value: '' }
  drawCount.value = 0
  kingCount.value = 0
  usedCards.value = []
  initDeck()
}

const getCardStyle = () => {
  const isRed = ['♥', '♦'].includes(currentCard.value.suit)
  return { color: isRed ? '#ef4444' : '#1e293b' }
}

// 小姐牌规则
const cardRules = {
  'A': {
    name: '指定喝酒',
    icon: '👆',
    desc: '指定任意一人喝一杯',
    color: '#ff6b6b'
  },
  '2': {
    name: '指定喝酒',
    icon: '👆',
    desc: '指定任意一人喝一杯',
    color: '#ff6b6b'
  },
  '3': {
    name: '自己喝',
    icon: '🍺',
    desc: '抽到3，自己喝一杯！',
    color: '#ffa726'
  },
  '4': {
    name: '拍地板',
    icon: '✋',
    desc: '所有人抢拍地板，最后拍的人喝酒！',
    color: '#66bb6a'
  },
  '5': {
    name: '小姐牌',
    icon: '👸',
    desc: '免死金牌！本轮免喝，也可以在之后别人让你喝时打出抵消。',
    color: '#e040fb'
  },
  '6': {
    name: '抓人',
    icon: '🤏',
    desc: '所有人立刻做一个手势（如摸鼻子），最后做的人喝酒！',
    color: '#42a5f5'
  },
  '7': {
    name: '抢7',
    icon: '7️⃣',
    desc: '从抽牌人开始依次报数，逢7或7的倍数拍手不能说，说错的人喝酒！',
    color: '#ab47bc'
  },
  '8': {
    name: '找搭档',
    icon: '🤝',
    desc: '指定一人做你的酒搭子，以后你喝酒时TA也要陪喝！',
    color: '#26c6da'
  },
  '9': {
    name: '左邻喝',
    icon: '⬅️',
    desc: '你左边的人喝一杯！',
    color: '#9ccc65'
  },
  '10': {
    name: '社交杯',
    icon: '🥂',
    desc: '所有人一起举杯，碰杯共饮！',
    color: '#ffd54f'
  },
  'J': {
    name: '制定规则',
    icon: '📜',
    desc: '制定一条新规则（如不能说"我"），违反者喝酒！规则持续到下一个J出现。',
    color: '#ff7043'
  },
  'Q': {
    name: '问题牌',
    icon: '❓',
    desc: '向任意一人提问，回答问题的人喝酒！（也可以反问，最终回答的人喝）',
    color: '#ec407a'
  },
  'K': {
    name: '倒酒牌',
    icon: '🍶',
    desc: '',
    color: '#ffd700'
  }
}

// 获取当前卡牌规则
const getCurrentRule = () => {
  if (!currentCard.value.value) return null
  const rule = cardRules[currentCard.value.value]
  if (!rule) return null

  // K的特殊处理
  if (currentCard.value.value === 'K') {
    if (kingCount.value < 4) {
      return {
        ...rule,
        desc: `往公杯里倒酒！（已出现 ${kingCount.value}/4 张K）`
      }
    } else {
      return {
        ...rule,
        desc: '第4张K！抽到的人把公杯里的酒全部喝掉！！！',
        color: '#ff1744'
      }
    }
  }
  return rule
}

// 规则列表数据
const rulesList = [
  { value: 'A', name: '指定喝酒', icon: '👆', brief: '指定一人喝' },
  { value: '2', name: '指定喝酒', icon: '👆', brief: '指定一人喝' },
  { value: '3', name: '自己喝', icon: '🍺', brief: '自己喝一杯' },
  { value: '4', name: '拍地板', icon: '✋', brief: '最后拍的人喝' },
  { value: '5', name: '小姐牌', icon: '👸', brief: '免死金牌' },
  { value: '6', name: '抓人', icon: '🤏', brief: '最后做手势的喝' },
  { value: '7', name: '抢7', icon: '7️⃣', brief: '逢7拍手' },
  { value: '8', name: '找搭档', icon: '🤝', brief: '指定酒搭子' },
  { value: '9', name: '左邻喝', icon: '⬅️', brief: '左边的人喝' },
  { value: '10', name: '社交杯', icon: '🥂', brief: '所有人碰杯' },
  { value: 'J', name: '制定规则', icon: '📜', brief: '定一条规则' },
  { value: 'Q', name: '问题牌', icon: '❓', brief: '提问，答者喝' },
  { value: 'K', name: '倒酒牌', icon: '🍶', brief: '往公杯倒酒' },
]

// 初始化
initDeck()
</script>

<template>
  <div class="miss-card-game">
    <!-- 顶部标题栏 -->
    <header class="mobile-header">
      <button class="back-btn" @click="$emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title">小姐牌</h1>
      <button class="rule-btn" @click="showRules = true">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"/>
          <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"/>
          <line x1="12" y1="17" x2="12.01" y2="17"/>
        </svg>
      </button>
    </header>

    <!-- 状态栏 -->
    <div class="status-bar status-bar-enter">
      <div class="status-item status-stagger" style="animation-delay: 0s">
        <span class="status-label">已抽</span>
        <span class="status-val">{{ drawCount }}</span>
      </div>
      <div class="status-item status-stagger" style="animation-delay: 0.06s">
        <span class="status-label">剩余</span>
        <span class="status-val">{{ remainingCards }}</span>
      </div>
      <div class="status-item king-status status-stagger" :class="{ 'king-danger': kingCount >= 4 }" style="animation-delay: 0.12s">
        <span class="status-label">K</span>
        <span class="status-val king-val">{{ kingCount }}/4</span>
      </div>
      <button class="reset-pill status-stagger" style="animation-delay: 0.18s" @click="reset">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 12"/>
          <path d="M3 3v9h9"/>
        </svg>
        洗牌
      </button>
    </div>

    <!-- 主游戏区域 -->
    <main class="game-area">
      <!-- 卡牌区域 -->
      <div class="card-section">
        <div
          class="card-wrapper"
          :class="{ flipping: isFlipping && !showCard, revealed: showCard }"
          @click="drawCard"
        >
          <div class="card-flipper" :class="{ flipped: showCard }">
            <!-- 背面 -->
            <div class="card card-back-face">
              <div class="back-outer">
                <div class="back-inner">
                  <div class="pattern-icon">🎴</div>
                  <div class="pattern-text">点击抽牌</div>
                  <div class="pattern-sub" v-if="remainingCards > 0">剩余 {{ remainingCards }} 张</div>
                </div>
              </div>
            </div>
            <!-- 正面 -->
            <div class="card card-front-face" :style="getCardStyle()">
              <span class="card-corner card-corner-top">
                <span class="corner-value">{{ currentCard.value }}</span>
                <span class="corner-suit">{{ currentCard.suit }}</span>
              </span>
              <div class="card-center">
                <div class="card-center-value">{{ currentCard.value }}</div>
                <div class="card-center-suit">{{ currentCard.suit }}</div>
              </div>
              <span class="card-corner card-corner-bottom">
                <span class="corner-value">{{ currentCard.value }}</span>
                <span class="corner-suit">{{ currentCard.suit }}</span>
              </span>
            </div>
          </div>
        </div>
      </div>

      <!-- 规则显示区域 -->
      <div v-if="showCard && !isDrawing && getCurrentRule()" class="rule-display">
        <div class="rule-card" :style="{ '--rule-color': getCurrentRule().color }">
          <div class="rule-header-row">
            <span class="rule-icon-big">{{ getCurrentRule().icon }}</span>
            <div class="rule-info">
              <div class="rule-name">{{ getCurrentRule().name }}</div>
              <div class="rule-card-label">{{ currentCard.value }}{{ currentCard.suit }}</div>
            </div>
          </div>
          <div class="rule-desc">{{ getCurrentRule().desc }}</div>
          <div v-if="currentCard.value === 'K' && kingCount >= 4" class="rule-danger">
            <span class="danger-icon">⚠️</span>
            <span>公杯时刻！全场最刺激的一杯！</span>
          </div>
        </div>
      </div>

      <!-- 抽牌按钮 -->
      <div class="action-area">
        <button
          class="draw-btn shimmer-btn"
          @click="drawCard"
          :disabled="isDrawing"
          :class="{ drawing: isDrawing }"
        >
          <span v-if="!isDrawing" class="btn-content">
            <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <rect x="2" y="4" width="20" height="16" rx="2"/>
              <path d="M12 4v16"/>
            </svg>
            <span>{{ showCard ? '再抽一张' : '抽牌' }}</span>
          </span>
          <span v-else class="btn-content">
            <svg class="spinning" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M12 2v4M12 18v4M4.93 4.93l2.83 2.83M16.24 16.24l2.83 2.83M2 12h4M18 12h4M4.93 19.07l2.83-2.83M16.24 7.76l2.83-2.83"/>
            </svg>
            <span>抽牌中...</span>
          </span>
        </button>
      </div>
    </main>

    <!-- 规则弹窗 (底部抽屉) -->
    <Transition name="sheet">
      <div v-if="showRules" class="sheet-overlay" @click.self="showRules = false">
        <div class="sheet-content">
          <div class="sheet-handle"></div>
          <div class="sheet-title">小姐牌规则</div>
          <div class="sheet-subtitle">每张牌都有对应的喝酒规则</div>

          <div class="rules-list">
            <div
              v-for="(rule, rIdx) in rulesList"
              :key="rule.value"
              class="rule-row rule-row-stagger"
              :style="{ animationDelay: rIdx * 0.04 + 's' }"
            >
              <div class="rule-val-badge">{{ rule.value }}</div>
              <div class="rule-icon-sm">{{ rule.icon }}</div>
              <div class="rule-text-area">
                <div class="rule-row-name">{{ rule.name }}</div>
                <div class="rule-row-brief">{{ rule.brief }}</div>
              </div>
            </div>
          </div>

          <div class="sheet-note">
            <span class="note-icon">💡</span>
            <span>提示：第4张K出现时，抽到的人要喝掉公杯里所有的酒！</span>
          </div>

          <button class="sheet-close-btn" @click="showRules = false">知道了</button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.miss-card-game {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: #0a0a14;
  background-image:
    radial-gradient(ellipse at 30% 0%, rgba(224, 64, 251, 0.08) 0%, transparent 50%),
    radial-gradient(ellipse at 70% 100%, rgba(102, 126, 234, 0.06) 0%, transparent 50%);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  color: #fff;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* ========== Header ========== */
.mobile-header {
  display: flex;
  align-items: center;
  padding: 10px 16px;
  padding-top: max(10px, env(safe-area-inset-top));
  background: rgba(10, 10, 20, 0.6);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  flex-shrink: 0;
}

.back-btn, .rule-btn {
  width: 42px; height: 42px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.85);
  cursor: pointer;
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}
.back-btn:active, .rule-btn:active {
  transform: scale(0.92);
  background: rgba(255, 255, 255, 0.12);
}

.header-title {
  flex: 1;
  text-align: center;
  font-size: 1.1rem;
  font-weight: 700;
  background: linear-gradient(135deg, #e040fb, #667eea);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* ========== Status Bar with entrance animation ========== */
@keyframes statusSlideIn {
  0% {
    opacity: 0;
    transform: translateY(-8px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

.status-stagger {
  animation: statusSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.status-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  flex-shrink: 0;
}

.status-item {
  display: flex; align-items: center; gap: 6px;
  padding: 6px 12px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 10px;
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.status-label {
  font-size: 0.72rem;
  color: rgba(255, 255, 255, 0.35);
  font-weight: 500;
}

.status-val {
  font-size: 0.9rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.7);
}

.king-status {
  border-color: rgba(255, 215, 0, 0.15);
  background: rgba(255, 215, 0, 0.06);
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

/* K 4/4 danger warning flash */
.king-danger {
  border-color: rgba(255, 23, 68, 0.5);
  background: rgba(255, 23, 68, 0.12);
  animation: kingDangerFlash 1s cubic-bezier(0.22, 1, 0.36, 1) infinite;
}

@keyframes kingDangerFlash {
  0%, 100% {
    border-color: rgba(255, 23, 68, 0.5);
    background: rgba(255, 23, 68, 0.12);
    box-shadow: 0 0 0 0 rgba(255, 23, 68, 0);
  }
  50% {
    border-color: rgba(255, 23, 68, 0.8);
    background: rgba(255, 23, 68, 0.2);
    box-shadow: 0 0 16px rgba(255, 23, 68, 0.25);
  }
}

.king-danger .king-val {
  color: #ff1744 !important;
  animation: kingValBlink 1s ease-in-out infinite;
}

@keyframes kingValBlink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.king-val {
  color: #ffd700;
}

.reset-pill {
  margin-left: auto;
  display: flex; align-items: center; gap: 4px;
  padding: 6px 12px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.75rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}
.reset-pill:active {
  transform: scale(0.95);
  background: rgba(255, 255, 255, 0.08);
}

/* ========== Game Area ========== */
.game-area {
  flex: 1;
  overflow-y: auto;
  padding: 8px 16px 20px;
  padding-bottom: max(20px, env(safe-area-inset-bottom));
  display: flex;
  flex-direction: column;
  -webkit-overflow-scrolling: touch;
}

/* ========== Card Section ========== */
.card-section {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 240px;
  perspective: 1200px;
}

.card-wrapper {
  cursor: pointer;
  transition: transform 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}
.card-wrapper:active { transform: scale(0.97); }

.card-wrapper.flipping .card-flipper {
  animation: cardLift 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes cardLift {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20px); }
}

/* ========== 3D Flip with dynamic shadows ========== */
.card-flipper {
  position: relative;
  width: 170px; height: 245px;
  transition: transform 0.7s cubic-bezier(0.22, 1, 0.36, 1);
  transform-style: preserve-3d;
}
.card-flipper.flipped { transform: rotateY(180deg); }

.card {
  position: absolute; top: 0; left: 0;
  width: 100%; height: 100%;
  border-radius: 16px;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  overflow: hidden;
  transition: box-shadow 0.7s cubic-bezier(0.22, 1, 0.36, 1);
}

.card-back-face {
  background: linear-gradient(135deg, #e040fb 0%, #7c4dff 50%, #667eea 100%);
  border: 1px solid rgba(255, 255, 255, 0.15);
  box-shadow: 0 12px 40px rgba(224, 64, 251, 0.2), 0 4px 16px rgba(0, 0, 0, 0.4);
  z-index: 2;
}

/* Dynamic shadow during flip - expand then contract */
.card-wrapper.flipping .card-back-face,
.card-wrapper.revealed .card-front-face {
  animation: flipShadow 0.7s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes flipShadow {
  0% {
    box-shadow: 0 12px 40px rgba(224, 64, 251, 0.2), 0 4px 16px rgba(0, 0, 0, 0.4);
  }
  40% {
    box-shadow: 0 24px 60px rgba(224, 64, 251, 0.35), 0 10px 30px rgba(0, 0, 0, 0.5);
  }
  100% {
    box-shadow: 0 12px 40px rgba(0, 0, 0, 0.3), 0 4px 16px rgba(0, 0, 0, 0.2);
  }
}

.card-front-face {
  background: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.08);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.3), 0 4px 16px rgba(0, 0, 0, 0.2);
  transform: rotateY(180deg);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.back-outer {
  width: 100%; height: 100%;
  display: flex; align-items: center; justify-content: center;
  background: repeating-linear-gradient(45deg, transparent, transparent 6px, rgba(255,255,255,0.04) 6px, rgba(255,255,255,0.04) 12px);
}

.back-inner {
  text-align: center;
  padding: 16px;
  border: 2px solid rgba(255, 255, 255, 0.15);
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.05);
}

.pattern-icon { font-size: 3.2rem; margin-bottom: 8px; }
.pattern-text { font-size: 0.95rem; color: rgba(255,255,255,0.7); font-weight: 600; }
.pattern-sub { font-size: 0.72rem; color: rgba(255,255,255,0.4); margin-top: 4px; }

/* Card front */
.card-corner {
  position: absolute;
  display: flex; flex-direction: column; align-items: center; line-height: 1;
}
.card-corner-top { top: 10px; left: 12px; }
.card-corner-bottom { bottom: 10px; right: 12px; transform: rotate(180deg); }
.corner-value { font-size: 0.9rem; font-weight: 800; }
.corner-suit { font-size: 0.75rem; margin-top: 1px; }

.card-center {
  display: flex; flex-direction: column; align-items: center; gap: 2px;
}
.card-center-value { font-size: 2.5rem; font-weight: 800; }
.card-center-suit { font-size: 3rem; }

/* ========== Rule Display ========== */
.rule-display {
  margin-top: 16px;
  animation: ruleSlideUp 0.45s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes ruleSlideUp {
  0% {
    opacity: 0;
    transform: translateY(24px) scale(0.97);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.rule-card {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 18px;
  padding: 18px;
  position: relative;
  overflow: hidden;
}

.rule-card::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0; height: 3px;
  background: var(--rule-color, #e040fb);
}

.rule-header-row {
  display: flex; align-items: center; gap: 14px;
  margin-bottom: 12px;
}

.rule-icon-big {
  width: 48px; height: 48px;
  display: flex; align-items: center; justify-content: center;
  font-size: 1.6rem;
  background: rgba(255, 255, 255, 0.06);
  border-radius: 14px;
}

.rule-info { flex: 1; }

.rule-name {
  font-size: 1.1rem;
  font-weight: 700;
  color: #fff;
  margin-bottom: 2px;
}

.rule-card-label {
  font-size: 0.78rem;
  color: rgba(255, 255, 255, 0.35);
  font-weight: 500;
}

.rule-desc {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.7);
  line-height: 1.55;
}

.rule-danger {
  margin-top: 12px;
  padding: 12px 14px;
  background: rgba(255, 23, 68, 0.1);
  border: 1px solid rgba(255, 23, 68, 0.2);
  border-radius: 12px;
  display: flex; align-items: center; gap: 8px;
  font-size: 0.85rem;
  color: #ff1744;
  font-weight: 600;
  animation: dangerPulse 1.5s cubic-bezier(0.22, 1, 0.36, 1) infinite;
}

@keyframes dangerPulse {
  0%, 100% {
    opacity: 1;
    box-shadow: 0 0 0 0 rgba(255, 23, 68, 0);
  }
  50% {
    opacity: 0.85;
    box-shadow: 0 0 16px rgba(255, 23, 68, 0.15);
  }
}

.danger-icon { font-size: 1.1rem; }

/* ========== Action Area ========== */
.action-area {
  margin-top: 16px;
  flex-shrink: 0;
}

/* Shimmer effect for buttons */
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

.draw-btn {
  width: 100%; height: 56px;
  display: flex; align-items: center; justify-content: center;
  background: linear-gradient(135deg, #e040fb 0%, #667eea 100%);
  border: none; border-radius: 16px;
  color: #fff;
  font-size: 1.05rem; font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 24px rgba(224, 64, 251, 0.25);
  transition: all 0.2s cubic-bezier(0.22, 1, 0.36, 1);
  -webkit-tap-highlight-color: transparent;
}

.draw-btn:active:not(.drawing) {
  transform: scale(0.97);
  box-shadow: 0 2px 12px rgba(224, 64, 251, 0.3);
}

.draw-btn.drawing {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-content {
  display: flex; align-items: center; gap: 10px;
}

.spinning {
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* ========== Rules Sheet ========== */
.sheet-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 100;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}

.sheet-content {
  width: 100%;
  max-width: 500px;
  max-height: 85vh;
  background: #15152a;
  border-radius: 24px 24px 0 0;
  padding: 16px 20px;
  padding-bottom: max(20px, env(safe-area-inset-bottom));
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

.sheet-handle {
  width: 36px; height: 4px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 2px;
  margin: 0 auto 16px;
}

.sheet-title {
  font-size: 1.2rem;
  font-weight: 700;
  text-align: center;
  margin-bottom: 4px;
}

.sheet-subtitle {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.35);
  text-align: center;
  margin-bottom: 20px;
}

.rules-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-bottom: 16px;
}

/* Staggered entrance for rule rows in drawer */
@keyframes ruleRowSlideIn {
  0% {
    opacity: 0;
    transform: translateX(-12px);
  }
  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

.rule-row-stagger {
  animation: ruleRowSlideIn 0.35s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.rule-row {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 12px;
}

.rule-val-badge {
  width: 30px; height: 30px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(224, 64, 251, 0.12);
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: 800;
  color: #e040fb;
  flex-shrink: 0;
}

.rule-icon-sm {
  font-size: 1.1rem;
  flex-shrink: 0;
}

.rule-text-area { flex: 1; }

.rule-row-name {
  font-size: 0.88rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.85);
}

.rule-row-brief {
  font-size: 0.72rem;
  color: rgba(255, 255, 255, 0.35);
  margin-top: 1px;
}

.sheet-note {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 12px;
  background: rgba(255, 215, 0, 0.06);
  border: 1px solid rgba(255, 215, 0, 0.12);
  border-radius: 12px;
  margin-bottom: 16px;
  font-size: 0.8rem;
  color: rgba(255, 215, 0, 0.8);
  line-height: 1.4;
}

.note-icon { font-size: 1rem; flex-shrink: 0; }

.sheet-close-btn {
  width: 100%; height: 50px;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 14px;
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}
.sheet-close-btn:active {
  transform: scale(0.97);
  background: rgba(255, 255, 255, 0.1);
}

/* Sheet transition */
.sheet-enter-active { transition: all 0.35s cubic-bezier(0.22, 1, 0.36, 1); }
.sheet-leave-active { transition: all 0.25s ease-in; }
.sheet-enter-from, .sheet-leave-to {
  opacity: 0;
}
.sheet-enter-from .sheet-content,
.sheet-leave-to .sheet-content {
  transform: translateY(100%);
}

/* ========== Responsive ========== */
@media (max-height: 700px) {
  .card-flipper { width: 145px; height: 210px; }
  .card-section { min-height: 210px; }
  .card-center-value { font-size: 2rem; }
  .card-center-suit { font-size: 2.5rem; }
  .pattern-icon { font-size: 2.5rem; }
}

@media (max-height: 600px) {
  .card-flipper { width: 120px; height: 175px; }
  .card-section { min-height: 175px; }
  .game-area { padding: 6px 12px 16px; }
  .status-bar { padding: 6px 12px; }
}
</style>
