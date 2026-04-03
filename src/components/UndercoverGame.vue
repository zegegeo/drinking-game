<script setup>
import { ref, computed, nextTick } from 'vue'

defineEmits(['goBack'])

const playerCount = ref(0)
const playerOptions = [3, 4, 5, 6, 7, 8]
const wordSet = ref('')
const undercoverWord = ref('卧底词')
const normalWord = ref('平民词')
const gamePhase = ref('setup') // 'setup', 'playing', 'result'
const currentPlayerIndex = ref(0)
const gameResults = ref([])
const showRules = ref(false)
const cardFlipped = ref(false)

const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(25)
  }
}

const wordSets = {
  '水果类': { normal: '苹果', undercover: '梨' },
  '动物类': { normal: '猫', undercover: '老虎' },
  '运动类': { normal: '篮球', undercover: '足球' },
  '职业类': { normal: '医生', undercover: '护士' },
  '饮料类': { normal: '可乐', undercover: '雪碧' },
  '食品类': { normal: '汉堡', undercover: '三明治' },
  '交通类': { normal: '地铁', undercover: '公交车' },
  '电器类': { normal: '冰箱', undercover: '空调' },
  '文具类': { normal: '铅笔', undercover: '钢笔' },
  '乐器类': { normal: '钢琴', undercover: '古筝' },
  '天气类': { normal: '下雨', undercover: '下雪' },
  '家具类': { normal: '沙发', undercover: '床' },
  '服饰类': { normal: '外套', undercover: '毛衣' },
  '建筑类': { normal: '医院', undercover: '学校' },
  '影视类': { normal: '电影', undercover: '电视剧' },
  '社交类': { normal: '微信', undercover: '微博' }
}

const canStartGame = computed(() => {
  return playerCount.value >= 3 && wordSet.value
})

const currentPlayer = computed(() => {
  return `玩家${currentPlayerIndex.value + 1}`
})

const players = computed(() => {
  return Array.from({ length: playerCount.value }, (_, i) => `玩家${i + 1}`)
})

const selectWordSet = (key) => {
  triggerHaptic()
  wordSet.value = key
  normalWord.value = wordSets[key].normal
  undercoverWord.value = wordSets[key].undercover
}

const startGame = () => {
  triggerHaptic()
  const undercoverIndex = Math.floor(Math.random() * playerCount.value)

  gameResults.value = players.value.map((name, index) => ({
    name,
    isUndercover: index === undercoverIndex,
    revealed: false
  }))

  currentPlayerIndex.value = 0
  cardFlipped.value = false
  gamePhase.value = 'playing'
}

const revealPlayer = () => {
  triggerHaptic()
  if (gameResults.value[currentPlayerIndex.value]) {
    gameResults.value[currentPlayerIndex.value].revealed = true
    cardFlipped.value = true
  }
}

const nextPlayer = () => {
  triggerHaptic()
  if (currentPlayerIndex.value < gameResults.value.length - 1) {
    cardFlipped.value = false
    nextTick(() => {
      currentPlayerIndex.value++
    })
  }
}

const showAllResults = () => {
  triggerHaptic()
  gameResults.value.forEach(r => r.revealed = true)
  gamePhase.value = 'result'
}

const resetGame = () => {
  triggerHaptic()
  gamePhase.value = 'setup'
  gameResults.value = []
  currentPlayerIndex.value = 0
  cardFlipped.value = false
}
</script>

<template>
  <div class="undercover-mobile">
    <!-- 背景装饰光点 -->
    <div class="bg-decoration">
      <div class="bg-orb bg-orb-1"></div>
      <div class="bg-orb bg-orb-2"></div>
      <div class="bg-orb bg-orb-3"></div>
    </div>

    <!-- 顶部标题栏 -->
    <header class="mobile-header">
      <button class="back-btn" @click="$emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title">谁是卧底</h1>
      <button class="rules-btn" @click="showRules = true">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"/>
          <path d="M9.09 9a3 3 0 015.83 1c0 2-3 3-3 3M12 17h.01"/>
        </svg>
      </button>
    </header>

    <!-- 主游戏区域 -->
    <main class="game-area">
      <!-- 设置阶段 -->
      <div v-if="gamePhase === 'setup'" class="setup-phase">
        <!-- 玩家人数选择 -->
        <div class="section-card">
          <div class="section-title">
            <span class="section-icon">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/>
                <circle cx="9" cy="7" r="4"/>
                <path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/>
              </svg>
            </span>
            选择玩家数量
          </div>
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
          <div v-if="playerCount > 0" class="player-preview">
            <div class="preview-chips">
              <span
                v-for="(p, idx) in players"
                :key="p"
                class="preview-chip"
                :style="{ animationDelay: (idx * 0.05) + 's' }"
              >{{ p }}</span>
            </div>
          </div>
        </div>

        <!-- 词语选择 -->
        <div class="section-card">
          <div class="section-title">
            <span class="section-icon">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M4 7V4h16v3M9 20h6M12 4v16"/>
              </svg>
            </span>
            选择词语类别
          </div>
          <div class="word-sets">
            <button
              v-for="(set, key) in wordSets"
              :key="key"
              class="word-set-btn"
              :class="{ active: wordSet === key }"
              @click="selectWordSet(key)"
            >
              {{ key }}
            </button>
          </div>
          <div v-if="wordSet" class="word-preview">
            <div class="word-item normal">
              <span class="word-label">平民词</span>
              <span class="word-value">{{ normalWord }}</span>
            </div>
            <div class="word-item undercover">
              <span class="word-label">卧底词</span>
              <span class="word-value">{{ undercoverWord }}</span>
            </div>
          </div>
        </div>

        <!-- 开始按钮 -->
        <button
          class="start-game-btn"
          :class="{ disabled: !canStartGame }"
          @click="startGame"
          :disabled="!canStartGame"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <polygon points="5 3 19 12 5 21 5 3"/>
          </svg>
          开始游戏
        </button>
      </div>

      <!-- 游戏进行中 -->
      <div v-else-if="gamePhase === 'playing'" class="playing-phase">
        <div class="phase-header">
          <div class="current-player">{{ currentPlayer }}</div>
          <div class="player-progress">
            <div class="progress-bar">
              <div class="progress-fill" :style="{ width: ((currentPlayerIndex + 1) / gameResults.length * 100) + '%' }"></div>
            </div>
            <div class="player-index">{{ currentPlayerIndex + 1 }} / {{ gameResults.length }}</div>
          </div>
        </div>

        <div class="card-area">
          <Transition name="card-flip" mode="out-in">
            <!-- 正面：未翻开 -->
            <div
              v-if="!cardFlipped"
              key="front"
              class="word-card card-front"
              @click="revealPlayer()"
            >
              <div class="card-mystery-icon">
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <circle cx="12" cy="12" r="10"/>
                  <path d="M9.09 9a3 3 0 015.83 1c0 2-3 3-3 3M12 17h.01"/>
                </svg>
              </div>
              <div class="tap-hint">点击翻开查看词语</div>
            </div>
            <!-- 背面：已翻开 -->
            <div
              v-else
              key="back"
              class="word-card card-back"
            >
              <div class="word-reveal-label">你的词语是</div>
              <div class="word-reveal-text">
                {{ gameResults[currentPlayerIndex]?.isUndercover ? undercoverWord : normalWord }}
              </div>
              <div class="word-role-hint">
                记住你的词语后请传给下一位
              </div>
            </div>
          </Transition>
        </div>

        <div class="playing-actions">
          <template v-if="gameResults[currentPlayerIndex]?.revealed">
            <button
              v-if="currentPlayerIndex < gameResults.length - 1"
              class="next-player-btn shimmer-btn"
              @click="nextPlayer"
            >
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <path d="M5 12h14M12 5l7 7-7 7"/>
              </svg>
              下一位玩家
            </button>
            <button class="show-results-btn" @click="showAllResults">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/>
                <circle cx="12" cy="12" r="3"/>
              </svg>
              揭晓答案
            </button>
          </template>
        </div>
      </div>

      <!-- 游戏结果 -->
      <div v-else-if="gamePhase === 'result'" class="result-phase">
        <div class="result-header">
          <div class="result-badge">
            <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="12" cy="8" r="7"/>
              <polyline points="8.21 13.89 7 23 12 20 17 23 15.79 13.88"/>
            </svg>
          </div>
          <div class="result-title">游戏结束</div>
          <div class="result-subtitle">卧底已经暴露!</div>
        </div>

        <div class="results-list">
          <div
            v-for="(result, index) in gameResults"
            :key="index"
            class="result-item"
            :class="{ undercover: result.isUndercover }"
            :style="{ animationDelay: (index * 0.1) + 's' }"
          >
            <div class="result-avatar" :class="{ 'avatar-undercover': result.isUndercover }">
              {{ result.name.charAt(0) }}
            </div>
            <div class="result-info">
              <div class="result-name">{{ result.name }}</div>
              <div class="result-word">{{ result.isUndercover ? undercoverWord : normalWord }}</div>
            </div>
            <div class="result-role-tag" :class="{ 'tag-undercover': result.isUndercover }">
              {{ result.isUndercover ? '卧底' : '平民' }}
            </div>
          </div>
        </div>

        <div class="result-actions">
          <button class="reset-btn shimmer-btn" @click="resetGame">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <polyline points="23 4 23 10 17 10"/>
              <path d="M20.49 15a9 9 0 11-2.12-9.36L23 10"/>
            </svg>
            再来一局
          </button>
        </div>
      </div>
    </main>

    <!-- 规则弹窗 - Bottom Sheet -->
    <Transition name="sheet">
      <div v-if="showRules" class="rules-overlay" @click="showRules = false">
        <div class="rules-sheet" @click.stop>
          <div class="sheet-handle"></div>
          <div class="rules-header">
            <h2>游戏规则</h2>
            <button class="close-btn" @click="showRules = false">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M18 6L6 18M6 6l12 12"/>
              </svg>
            </button>
          </div>
          <div class="rules-body">
            <div class="rule-item">
              <div class="rule-number">1</div>
              <div class="rule-text">所有玩家依次查看自己的词语，只有一人的词语与其他人不同（卧底词）</div>
            </div>
            <div class="rule-item">
              <div class="rule-number">2</div>
              <div class="rule-text">从第二位玩家开始，每人描述自己的词语（不能直接说出词语）</div>
            </div>
            <div class="rule-item">
              <div class="rule-number">3</div>
              <div class="rule-text">每轮描述后，玩家投票选出认为的卧底</div>
            </div>
            <div class="rule-item">
              <div class="rule-number">4</div>
              <div class="rule-text">卧底被投出则平民获胜，只剩两人时卧底获胜</div>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
/* ===== 全局容器 ===== */
.undercover-mobile {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: #0a0a14;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}

/* ===== 背景装饰光点 ===== */
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
  opacity: 0.35;
  animation: orbFloat 12s ease-in-out infinite;
}

.bg-orb-1 {
  width: 200px; height: 200px;
  background: radial-gradient(circle, rgba(102, 126, 234, 0.3), transparent 70%);
  top: 10%; left: -5%;
}
.bg-orb-2 {
  width: 160px; height: 160px;
  background: radial-gradient(circle, rgba(118, 75, 162, 0.25), transparent 70%);
  top: 50%; right: -8%;
  animation-delay: -4s;
}
.bg-orb-3 {
  width: 140px; height: 140px;
  background: radial-gradient(circle, rgba(141, 162, 251, 0.2), transparent 70%);
  bottom: 15%; left: 20%;
  animation-delay: -8s;
}

@keyframes orbFloat {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(15px, -20px) scale(1.05); }
  66% { transform: translate(-10px, 15px) scale(0.95); }
}

/* ===== Header ===== */
.mobile-header {
  display: flex;
  align-items: center;
  padding: 10px 16px;
  background: rgba(10, 10, 20, 0.75);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  z-index: 50;
  flex-shrink: 0;
  position: relative;
}

.back-btn {
  width: 42px; height: 42px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.85);
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.back-btn:active {
  background: rgba(255, 255, 255, 0.12);
  transform: scale(0.93);
}

.header-title {
  flex: 1; text-align: center;
  font-size: 1.1rem; font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #a78bfa 50%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: 0.5px;
  margin: 0;
}

.rules-btn {
  width: 42px; height: 42px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.85);
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.rules-btn:active {
  background: rgba(255, 255, 255, 0.12);
  transform: scale(0.93);
}

/* ===== 主游戏区域 ===== */
.game-area {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 20px 16px;
  padding-bottom: calc(32px + env(safe-area-inset-bottom));
  -webkit-overflow-scrolling: touch;
  position: relative;
  z-index: 1;
}

/* ===== 设置阶段 ===== */
.setup-phase {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.section-card {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.07);
  border-radius: 18px;
  padding: 18px;
  animation: sectionFadeIn 0.5s cubic-bezier(0.22, 1, 0.36, 1) both;
}
.section-card:nth-child(1) { animation-delay: 0s; }
.section-card:nth-child(2) { animation-delay: 0.1s; }

@keyframes sectionFadeIn {
  from { opacity: 0; transform: translateY(16px); }
  to { opacity: 1; transform: translateY(0); }
}

.section-title {
  font-size: 0.92rem; font-weight: 600;
  color: rgba(255, 255, 255, 0.75);
  margin-bottom: 14px;
  display: flex; align-items: center; gap: 8px;
}

.section-icon {
  display: flex; align-items: center;
  color: rgba(102, 126, 234, 0.8);
}

/* ===== 玩家数量选择 ===== */
.player-pills {
  display: flex; flex-wrap: wrap; gap: 10px;
  margin-bottom: 14px;
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
  background: linear-gradient(135deg, rgba(102,126,234,0.2), rgba(118,75,162,0.2));
  border-color: rgba(102,126,234,0.5);
  color: #8da2fb;
  font-weight: 600;
  box-shadow: 0 0 16px rgba(102,126,234,0.15);
}

.pill-stagger {
  animation: pillStaggerIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes pillStaggerIn {
  from { opacity: 0; transform: translateY(10px) scale(0.9); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}

.player-preview { margin-top: 4px; }

.preview-chips {
  display: flex; flex-wrap: wrap; gap: 8px;
}

.preview-chip {
  padding: 6px 14px;
  background: rgba(102,126,234,0.08);
  border: 1px solid rgba(102,126,234,0.15);
  border-radius: 12px;
  font-size: 0.82rem; color: rgba(255,255,255,0.7); font-weight: 500;
  animation: chipFadeIn 0.3s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes chipFadeIn {
  from { opacity: 0; transform: scale(0.85); }
  to { opacity: 1; transform: scale(1); }
}

/* ===== 词语选择 ===== */
.word-sets {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
  margin-bottom: 14px;
}

.word-set-btn {
  padding: 11px 6px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.82rem; font-weight: 500;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  text-align: center;
}
.word-set-btn.active {
  background: rgba(102, 126, 234, 0.15);
  border-color: rgba(102, 126, 234, 0.5);
  color: #8da2fb;
  box-shadow: 0 0 20px rgba(102, 126, 234, 0.15);
}
.word-set-btn:active { transform: scale(0.96); }

.word-preview {
  display: flex; flex-direction: column; gap: 8px;
  animation: fadeSlideUp 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes fadeSlideUp {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

.word-item {
  padding: 12px 14px; border-radius: 12px;
  display: flex; justify-content: space-between; align-items: center;
}
.word-item.normal {
  background: rgba(46, 204, 113, 0.08);
  border: 1px solid rgba(46, 204, 113, 0.15);
}
.word-item.undercover {
  background: rgba(255, 107, 107, 0.08);
  border: 1px solid rgba(255, 107, 107, 0.15);
}
.word-label {
  font-size: 0.82rem; color: rgba(255, 255, 255, 0.45); font-weight: 500;
}
.word-value { font-size: 1rem; font-weight: 700; letter-spacing: 0.5px; }
.word-item.normal .word-value { color: #2ecc71; }
.word-item.undercover .word-value { color: #ff6b6b; }

/* ===== 开始按钮 ===== */
.start-game-btn {
  width: 100%; padding: 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none; border-radius: 18px;
  color: #fff; font-size: 1.05rem; font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 24px rgba(102, 126, 234, 0.35);
  display: flex; align-items: center; justify-content: center; gap: 8px;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  position: relative; overflow: hidden;
}
.start-game-btn::after {
  content: '';
  position: absolute;
  top: 0; left: -100%; width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.15), transparent);
  animation: shimmerBtn 3s ease-in-out infinite;
}
@keyframes shimmerBtn {
  0% { left: -100%; }
  50%, 100% { left: 100%; }
}
.start-game-btn:active:not(.disabled) {
  transform: scale(0.97);
}
.start-game-btn.disabled {
  opacity: 0.35; cursor: not-allowed; box-shadow: none;
}
.start-game-btn.disabled::after { animation: none; }

/* ===== Shimmer 按钮 ===== */
.shimmer-btn { position: relative; overflow: hidden; }
.shimmer-btn::after {
  content: '';
  position: absolute;
  top: 0; left: -100%; width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  animation: shimmerBtn 2.5s ease-in-out infinite;
}

/* ===== 游戏进行中 ===== */
.playing-phase {
  display: flex;
  flex-direction: column;
  min-height: calc(100vh - 120px);
}

.phase-header {
  text-align: center;
  margin-bottom: 20px;
  animation: fadeSlideUp 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

.current-player {
  font-size: 1.5rem; font-weight: 800;
  color: #fff; margin-bottom: 12px;
}

.player-progress {
  display: flex; flex-direction: column;
  align-items: center; gap: 8px;
}

.progress-bar {
  width: 140px; height: 4px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 2px; overflow: hidden;
}
.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea, #764ba2);
  border-radius: 2px;
  transition: width 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}
.player-index {
  font-size: 0.82rem; color: rgba(255, 255, 255, 0.4); font-weight: 500;
}

/* ===== 卡片区域（无3D翻转，纯切换动画） ===== */
.card-area {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  min-height: 220px;
  max-height: 320px;
}

.word-card {
  width: 100%;
  max-width: 320px;
  min-height: 220px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 22px;
  padding: 28px 20px;
  box-sizing: border-box;
}

.card-front {
  background: rgba(255, 255, 255, 0.04);
  border: 1.5px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
  cursor: pointer;
}

.card-front:active {
  transform: scale(0.97);
}

.card-mystery-icon {
  color: rgba(255, 255, 255, 0.2);
  margin-bottom: 16px;
  animation: floatIcon 3s ease-in-out infinite;
}

@keyframes floatIcon {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
}

.tap-hint {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.35);
  animation: pulseFade 2.5s ease-in-out infinite;
}

@keyframes pulseFade {
  0%, 100% { opacity: 0.35; }
  50% { opacity: 0.7; }
}

.card-back {
  background: linear-gradient(145deg, rgba(102, 126, 234, 0.15), rgba(118, 75, 162, 0.15));
  border: 1.5px solid rgba(102, 126, 234, 0.3);
  box-shadow: 0 12px 40px rgba(102, 126, 234, 0.2), 0 0 60px rgba(102, 126, 234, 0.08);
}

.word-reveal-label {
  font-size: 0.85rem; color: rgba(255, 255, 255, 0.45);
  margin-bottom: 16px; font-weight: 500;
}

.word-reveal-text {
  font-size: 2.2rem; font-weight: 800;
  background: linear-gradient(135deg, #667eea 0%, #a78bfa 50%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: 2px;
}

.word-role-hint {
  margin-top: 20px;
  font-size: 0.78rem; color: rgba(255, 255, 255, 0.3);
  padding: 6px 14px;
  background: rgba(255, 255, 255, 0.04);
  border-radius: 20px;
}

/* ===== 卡片切换 Transition ===== */
.card-flip-enter-active {
  animation: cardFlipIn 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

.card-flip-leave-active {
  animation: cardFlipOut 0.25s ease-in;
}

@keyframes cardFlipIn {
  0% { opacity: 0; transform: scale(0.85) rotateY(8deg); }
  100% { opacity: 1; transform: scale(1) rotateY(0deg); }
}

@keyframes cardFlipOut {
  0% { opacity: 1; transform: scale(1) rotateY(0deg); }
  100% { opacity: 0; transform: scale(0.85) rotateY(-8deg); }
}

/* ===== 游戏操作按钮 ===== */
.playing-actions {
  display: flex; flex-direction: column;
  gap: 10px; flex-shrink: 0;
  position: relative;
  z-index: 10;
}

.next-player-btn,
.show-results-btn {
  width: 100%; padding: 15px;
  border: none; border-radius: 18px;
  font-size: 0.95rem; font-weight: 700;
  cursor: pointer;
  display: flex; align-items: center; justify-content: center; gap: 8px;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  animation: actionBtnIn 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes actionBtnIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

.next-player-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}
.next-player-btn:active { transform: scale(0.97); }

.show-results-btn {
  background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
  color: #fff;
  box-shadow: 0 4px 20px rgba(255, 107, 107, 0.25);
}
.show-results-btn:active { transform: scale(0.97); }

/* ===== 游戏结果 ===== */
.result-phase {
  display: flex; flex-direction: column;
  min-height: calc(100vh - 120px);
}

.result-header {
  text-align: center; margin-bottom: 24px;
  animation: resultHeaderIn 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes resultHeaderIn {
  0% { opacity: 0; transform: scale(0.8); }
  60% { transform: scale(1.05); }
  100% { opacity: 1; transform: scale(1); }
}

.result-badge {
  width: 56px; height: 56px;
  margin: 0 auto 14px;
  display: flex; align-items: center; justify-content: center;
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.15), rgba(118, 75, 162, 0.15));
  border: 1px solid rgba(102, 126, 234, 0.25);
  border-radius: 18px;
  color: #8da2fb;
}

.result-title {
  font-size: 1.4rem; font-weight: 800; color: #fff; margin-bottom: 4px;
}
.result-subtitle {
  font-size: 0.85rem; color: rgba(255, 255, 255, 0.4);
}

.results-list {
  flex: 1;
  display: flex; flex-direction: column; gap: 8px;
  overflow-y: auto;
  margin-bottom: 20px;
}

.result-item {
  display: flex; align-items: center; gap: 12px;
  padding: 13px 14px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.07);
  border-radius: 16px;
  animation: resultItemReveal 0.5s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes resultItemReveal {
  0% { opacity: 0; transform: scale(0.9) translateY(10px); }
  60% { transform: scale(1.02) translateY(-2px); }
  100% { opacity: 1; transform: scale(1) translateY(0); }
}

.result-item.undercover {
  background: rgba(255, 107, 107, 0.08);
  border-color: rgba(255, 107, 107, 0.2);
  box-shadow: 0 0 24px rgba(255, 107, 107, 0.08);
  animation: resultItemReveal 0.5s cubic-bezier(0.22, 1, 0.36, 1) both,
             undercoverGlow 2s ease-in-out infinite 0.6s;
}

@keyframes undercoverGlow {
  0%, 100% { box-shadow: 0 0 20px rgba(255, 107, 107, 0.08); }
  50% { box-shadow: 0 0 30px rgba(255, 107, 107, 0.18); }
}

.result-avatar {
  width: 42px; height: 42px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(46, 204, 113, 0.12);
  border: 1px solid rgba(46, 204, 113, 0.2);
  border-radius: 13px;
  font-size: 1rem; font-weight: 700; color: #2ecc71;
  flex-shrink: 0;
}
.result-avatar.avatar-undercover {
  background: rgba(255, 107, 107, 0.12);
  border-color: rgba(255, 107, 107, 0.2);
  color: #ff6b6b;
  animation: undercoverAvatarPulse 1.5s ease-in-out infinite;
}

@keyframes undercoverAvatarPulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(255, 107, 107, 0.3); }
  50% { box-shadow: 0 0 0 6px rgba(255, 107, 107, 0); }
}

.result-info { flex: 1; min-width: 0; }
.result-name {
  font-size: 0.95rem; font-weight: 700;
  color: rgba(255, 255, 255, 0.9); margin-bottom: 2px;
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}
.result-word { font-size: 0.78rem; color: rgba(255, 255, 255, 0.35); }

.result-role-tag {
  padding: 5px 12px;
  background: rgba(46, 204, 113, 0.12);
  border: 1px solid rgba(46, 204, 113, 0.2);
  border-radius: 20px;
  font-size: 0.78rem; font-weight: 700; color: #2ecc71;
  flex-shrink: 0;
}
.result-role-tag.tag-undercover {
  background: rgba(255, 107, 107, 0.15);
  border-color: rgba(255, 107, 107, 0.3);
  color: #ff6b6b;
  box-shadow: 0 0 12px rgba(255, 107, 107, 0.15);
}

.result-actions {
  flex-shrink: 0;
}

.reset-btn {
  width: 100%; padding: 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none; border-radius: 18px;
  color: #fff; font-size: 1.05rem; font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 24px rgba(102, 126, 234, 0.3);
  display: flex; align-items: center; justify-content: center; gap: 8px;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.reset-btn:active { transform: scale(0.97); }

/* ===== 规则弹窗 - Bottom Sheet ===== */
.rules-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  z-index: 200;
  display: flex; align-items: flex-end; justify-content: center;
}

.rules-sheet {
  width: 100%; max-width: 500px;
  background: #13131f;
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-bottom: none;
  border-radius: 24px 24px 0 0;
  padding: 10px 20px 20px;
  padding-bottom: calc(20px + env(safe-area-inset-bottom));
  max-height: 80vh; overflow-y: auto;
}

.sheet-handle {
  width: 36px; height: 4px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 2px;
  margin: 0 auto 16px;
}

.rules-header {
  display: flex; justify-content: space-between;
  align-items: center; margin-bottom: 20px;
}
.rules-header h2 {
  font-size: 1.15rem; font-weight: 800;
  color: #fff; margin: 0;
}

.close-btn {
  width: 34px; height: 34px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.close-btn:active {
  transform: scale(0.93);
  background: rgba(255, 255, 255, 0.1);
}

.rules-body {
  display: flex; flex-direction: column; gap: 14px;
}
.rule-item { display: flex; gap: 12px; align-items: flex-start; }
.rule-number {
  width: 28px; height: 28px;
  display: flex; align-items: center; justify-content: center;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border-radius: 9px;
  font-size: 0.82rem; font-weight: 800; color: #fff;
  flex-shrink: 0;
}
.rule-text {
  flex: 1; font-size: 0.88rem;
  color: rgba(255, 255, 255, 0.7);
  line-height: 1.6; padding-top: 3px;
}

/* ===== Transition for Bottom Sheet ===== */
.sheet-enter-active,
.sheet-leave-active {
  transition: opacity 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}
.sheet-enter-active .rules-sheet {
  transition: transform 0.35s cubic-bezier(0.22, 1, 0.36, 1);
}
.sheet-leave-active .rules-sheet {
  transition: transform 0.25s ease-in;
}
.sheet-enter-from { opacity: 0; }
.sheet-enter-from .rules-sheet { transform: translateY(100%); }
.sheet-leave-to { opacity: 0; }
.sheet-leave-to .rules-sheet { transform: translateY(100%); }

/* ===== 小屏适配 ===== */
@media (max-height: 680px) {
  .word-card { min-height: 180px; }
  .word-reveal-text { font-size: 1.8rem; }
  .current-player { font-size: 1.3rem; }
  .phase-header { margin-bottom: 14px; }
}

@media (min-width: 500px) {
  .game-area { max-width: 440px; margin: 0 auto; }
}

.game-area::-webkit-scrollbar { width: 0; display: none; }
</style>
