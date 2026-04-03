<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

defineEmits(['goBack'])

// 游戏阶段: setup, playing, result
const phase = ref('setup')

// 设置阶段
const playerCount = ref(2)

// 游戏阶段
const currentPlayerIndex = ref(0)
const gameState = ref('idle') // idle, waiting, ready, go, foul, done
const reactionTime = ref(0)
const results = ref([]) // { name, time, foul }
const waitTimer = ref(null)
const startTimestamp = ref(0)
const countdownActive = ref(false)

// 触觉反馈
const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(25)
  }
}

// 监听玩家数量变化
const setPlayerCount = (n) => {
  playerCount.value = n
  triggerHaptic()
}

// 开始游戏
const startGame = () => {
  results.value = []
  currentPlayerIndex.value = 0
  phase.value = 'playing'
  gameState.value = 'idle'
}

// 当前玩家名字
const currentPlayerName = computed(() => {
  return `玩家${currentPlayerIndex.value + 1}`
})

// 排行榜（按反应时间排序，犯规排最后）
const leaderboard = computed(() => {
  const sorted = [...results.value].sort((a, b) => {
    if (a.foul && !b.foul) return 1
    if (!a.foul && b.foul) return -1
    if (a.foul && b.foul) return 0
    return a.time - b.time
  })
  return sorted
})

// 最慢的人（非犯规中最慢 + 所有犯规的人）
const losers = computed(() => {
  const names = []
  const nonFoul = leaderboard.value.filter(r => !r.foul)
  const fouls = leaderboard.value.filter(r => r.foul)
  if (nonFoul.length > 0) {
    names.push(nonFoul[nonFoul.length - 1].name)
  }
  fouls.forEach(r => {
    if (!names.includes(r.name)) {
      names.push(r.name)
    }
  })
  return names
})

// 开始当前玩家的回合
const startRound = () => {
  triggerHaptic()
  gameState.value = 'waiting'
  countdownActive.value = true

  // 随机 1-5 秒后变绿
  const delay = 1000 + Math.random() * 4000
  waitTimer.value = setTimeout(() => {
    gameState.value = 'go'
    startTimestamp.value = performance.now()
    countdownActive.value = false
    triggerHaptic()
  }, delay)
}

// 玩家点击屏幕
const handleTap = () => {
  if (gameState.value === 'idle') {
    // 还没开始，不处理
    return
  }

  if (gameState.value === 'waiting') {
    // 提前点击 - 犯规！
    clearTimeout(waitTimer.value)
    waitTimer.value = null
    countdownActive.value = false
    gameState.value = 'foul'
    triggerHaptic()
    // 记录犯规结果
    results.value.push({
      name: currentPlayerName.value,
      time: 0,
      foul: true
    })
    return
  }

  if (gameState.value === 'go') {
    // 正常点击，计算反应时间
    const elapsed = Math.round(performance.now() - startTimestamp.value)
    reactionTime.value = elapsed
    gameState.value = 'done'
    triggerHaptic()
    results.value.push({
      name: currentPlayerName.value,
      time: elapsed,
      foul: false
    })
    return
  }
}

// 下一个玩家
const nextPlayer = () => {
  triggerHaptic()
  if (currentPlayerIndex.value < playerCount.value - 1) {
    currentPlayerIndex.value++
    gameState.value = 'idle'
  } else {
    // 所有人都测完了
    phase.value = 'result'
  }
}

// 反应速度评价
const getReactionGrade = (ms) => {
  if (ms < 200) return { text: '超人级!', color: '#00e676' }
  if (ms < 300) return { text: '很快!', color: '#76ff03' }
  if (ms < 400) return { text: '不错', color: '#ffeb3b' }
  if (ms < 500) return { text: '一般', color: '#ffa726' }
  return { text: '太慢了...', color: '#ff5252' }
}

// 重新开始
const restart = () => {
  triggerHaptic()
  phase.value = 'setup'
  gameState.value = 'idle'
  results.value = []
  currentPlayerIndex.value = 0
  reactionTime.value = 0
  if (waitTimer.value) {
    clearTimeout(waitTimer.value)
    waitTimer.value = null
  }
}

// 再来一局（保持玩家不变）
const playAgain = () => {
  triggerHaptic()
  phase.value = 'playing'
  gameState.value = 'idle'
  results.value = []
  currentPlayerIndex.value = 0
  reactionTime.value = 0
}

onMounted(() => {
  document.body.style.overflow = 'hidden'
})

onUnmounted(() => {
  document.body.style.overflow = ''
  if (waitTimer.value) {
    clearTimeout(waitTimer.value)
  }
})
</script>

<template>
  <div class="reaction-game">
    <!-- 顶部标题栏 -->
    <header class="top-bar">
      <button class="icon-btn" @click="phase === 'setup' ? $emit('goBack') : restart()">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="title">手速挑战</h1>
      <div class="header-spacer"></div>
    </header>

    <!-- ============ 设置阶段 ============ -->
    <main v-if="phase === 'setup'" class="main-area">
      <div class="setup-intro">
        <div class="intro-icon">
          <svg width="56" height="56" viewBox="0 0 24 24" fill="none" stroke="url(#introGrad)" stroke-width="1.5">
            <defs>
              <linearGradient id="introGrad" x1="0" y1="0" x2="24" y2="24">
                <stop offset="0%" stop-color="#ff6b6b"/>
                <stop offset="100%" stop-color="#ffa726"/>
              </linearGradient>
            </defs>
            <circle cx="12" cy="12" r="10"/>
            <polyline points="12 6 12 12 16 14"/>
          </svg>
        </div>
        <div class="intro-title">反应力测试</div>
        <div class="intro-desc">看谁的手速最快，最慢的人喝酒!</div>
      </div>

      <!-- 玩家数量选择 -->
      <div class="section-block">
        <div class="section-label">玩家数量</div>
        <div class="count-pills">
          <button
            v-for="n in 7"
            :key="n + 1"
            class="pill pill-stagger"
            :class="{ active: playerCount === n + 1 }"
            :style="{ animationDelay: n * 0.06 + 's' }"
            @click="setPlayerCount(n + 1)"
          >
            {{ n + 1 }}
          </button>
        </div>
      </div>

      <button class="start-btn shimmer-btn" @click="startGame">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <polygon points="5 3 19 12 5 21 5 3"/>
        </svg>
        开始游戏
      </button>

      <!-- 规则说明 -->
      <div class="rules-card">
        <div class="rules-title">游戏规则</div>
        <div class="rule-item">
          <span class="rule-dot waiting-dot"></span>
          <span>屏幕变橙色时耐心等待</span>
        </div>
        <div class="rule-item">
          <span class="rule-dot go-dot"></span>
          <span>变绿色后尽快点击屏幕</span>
        </div>
        <div class="rule-item">
          <span class="rule-dot foul-dot"></span>
          <span>提前点击 = 犯规罚酒</span>
        </div>
        <div class="rule-item">
          <span class="rule-dot slow-dot"></span>
          <span>反应最慢的人喝酒</span>
        </div>
      </div>
    </main>

    <!-- ============ 游戏阶段 ============ -->
    <div v-else-if="phase === 'playing'" class="game-fullscreen" @click="handleTap">

      <!-- idle: 等待开始 -->
      <div v-if="gameState === 'idle'" class="state-idle" @click.stop>
        <div class="idle-card">
          <div class="idle-round">第 {{ currentPlayerIndex + 1 }}/{{ playerCount }} 轮 - {{ currentPlayerName }}</div>
          <div class="idle-player">{{ currentPlayerName }}</div>
          <div class="idle-hint">准备好了吗?</div>
          <button class="idle-go-btn" @click="startRound">
            <span>开始</span>
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <path d="M5 12h14M12 5l7 7-7 7"/>
            </svg>
          </button>

          <!-- 已完成的玩家进度 -->
          <div v-if="results.length > 0" class="progress-dots">
            <div
              v-for="(r, idx) in results"
              :key="idx"
              class="p-dot"
              :class="{ foul: r.foul }"
            >
              <span class="p-dot-name">{{ r.name }}</span>
              <span class="p-dot-time">{{ r.foul ? '犯规' : r.time + 'ms' }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- waiting: 等待变绿 -->
      <div v-else-if="gameState === 'waiting'" class="state-waiting">
        <div class="waiting-content">
          <div class="waiting-dots">
            <span class="w-dot" style="animation-delay: 0s"></span>
            <span class="w-dot" style="animation-delay: 0.2s"></span>
            <span class="w-dot" style="animation-delay: 0.4s"></span>
          </div>
          <div class="waiting-text">等待...</div>
          <div class="waiting-sub">变绿后立即点击!</div>
          <div class="waiting-warn">提前点击会犯规哦</div>
        </div>
      </div>

      <!-- go: 快点击! -->
      <div v-else-if="gameState === 'go'" class="state-go">
        <div class="go-content">
          <div class="go-icon">
            <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2">
              <path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"/>
            </svg>
          </div>
          <div class="go-text">点击!</div>
          <div class="go-sub">快! 快! 快!</div>
        </div>
      </div>

      <!-- foul: 犯规 -->
      <div v-if="gameState === 'foul'" class="state-foul" @click.stop>
        <div class="foul-content">
          <div class="foul-icon">
            <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2">
              <circle cx="12" cy="12" r="10"/>
              <path d="M15 9l-6 6M9 9l6 6"/>
            </svg>
          </div>
          <div class="foul-title">犯规!</div>
          <div class="foul-player">{{ currentPlayerName }}</div>
          <div class="foul-penalty">罚喝一杯</div>
          <button class="foul-next-btn" @click="nextPlayer">
            {{ currentPlayerIndex < playerCount - 1 ? '下一位' : '查看排行' }}
          </button>
        </div>
      </div>

      <!-- done: 显示结果 -->
      <div v-if="gameState === 'done'" class="state-done" @click.stop>
        <div class="done-content">
          <div class="done-player">{{ currentPlayerName }}</div>
          <div class="done-time" :style="{ color: getReactionGrade(reactionTime).color }">
            {{ reactionTime }}
            <span class="done-unit">ms</span>
          </div>
          <div class="done-grade" :style="{ color: getReactionGrade(reactionTime).color }">
            {{ getReactionGrade(reactionTime).text }}
          </div>
          <div class="done-bar-wrap">
            <div class="done-bar-bg">
              <div
                class="done-bar-fill"
                :style="{
                  width: `${Math.min(100, (reactionTime / 600) * 100)}%`,
                  background: getReactionGrade(reactionTime).color
                }"
              ></div>
            </div>
            <div class="done-bar-labels">
              <span>0ms</span>
              <span>200ms</span>
              <span>400ms</span>
              <span>600ms+</span>
            </div>
          </div>
          <button class="done-next-btn" @click="nextPlayer">
            {{ currentPlayerIndex < playerCount - 1 ? '下一位' : '查看排行' }}
          </button>
        </div>
      </div>
    </div>

    <!-- ============ 结果阶段 ============ -->
    <main v-else-if="phase === 'result'" class="main-area">
      <div class="result-header">
        <div class="result-trophy">
          <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="url(#trophyGrad)" stroke-width="1.5">
            <defs>
              <linearGradient id="trophyGrad" x1="0" y1="0" x2="24" y2="24">
                <stop offset="0%" stop-color="#ffd700"/>
                <stop offset="100%" stop-color="#ffa726"/>
              </linearGradient>
            </defs>
            <path d="M6 9H4a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/>
            <path d="M18 9h2a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2h-2"/>
            <path d="M6 4h12v6a6 6 0 0 1-12 0V4z"/>
            <path d="M12 16v2"/>
            <path d="M8 20h8"/>
          </svg>
        </div>
        <div class="result-title">排行榜</div>
      </div>

      <div class="leaderboard">
        <div
          v-for="(entry, index) in leaderboard"
          :key="index"
          class="lb-row lb-row-stagger"
          :style="{ animationDelay: index * 0.08 + 's' }"
          :class="{
            'rank-1': index === 0 && !entry.foul,
            'is-loser': losers.includes(entry.name),
            'is-foul': entry.foul
          }"
        >
          <div class="lb-rank">
            <span v-if="!entry.foul" class="rank-num">{{ index + 1 }}</span>
            <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#ff5252" stroke-width="2">
              <circle cx="12" cy="12" r="10"/>
              <path d="M15 9l-6 6M9 9l6 6"/>
            </svg>
          </div>
          <div class="lb-info">
            <div class="lb-name">{{ entry.name }}</div>
            <div v-if="entry.foul" class="lb-foul-tag">犯规 - 罚酒</div>
            <div v-else class="lb-time-text">{{ entry.time }}ms</div>
          </div>
          <div class="lb-badge-area">
            <span v-if="index === 0 && !entry.foul" class="lb-badge fastest">最快</span>
            <span v-if="losers.includes(entry.name)" class="lb-badge drink">喝酒!</span>
          </div>
        </div>
      </div>

      <!-- 统计信息 -->
      <div class="stats-card">
        <div class="stat-item stat-pop" style="animation-delay: 0s" v-if="leaderboard.filter(e => !e.foul).length > 0">
          <span class="stat-label">最快反应</span>
          <span class="stat-value stat-num-pop">{{ leaderboard.filter(e => !e.foul)[0]?.time || '-' }}ms</span>
        </div>
        <div class="stat-item stat-pop" style="animation-delay: 0.08s" v-if="leaderboard.filter(e => !e.foul).length > 0">
          <span class="stat-label">最慢反应</span>
          <span class="stat-value stat-num-pop" style="animation-delay: 0.15s">{{ leaderboard.filter(e => !e.foul).slice(-1)[0]?.time || '-' }}ms</span>
        </div>
        <div class="stat-item stat-pop" style="animation-delay: 0.16s" v-if="leaderboard.filter(e => !e.foul).length > 0">
          <span class="stat-label">平均反应</span>
          <span class="stat-value stat-num-pop" style="animation-delay: 0.25s">{{ Math.round(leaderboard.filter(e => !e.foul).reduce((s, e) => s + e.time, 0) / leaderboard.filter(e => !e.foul).length) }}ms</span>
        </div>
        <div class="stat-item stat-pop" style="animation-delay: 0.24s">
          <span class="stat-label">犯规人数</span>
          <span class="stat-value foul-val stat-num-pop" style="animation-delay: 0.35s">{{ leaderboard.filter(e => e.foul).length }}</span>
        </div>
      </div>

      <!-- 操作按钮 -->
      <div class="result-actions">
        <button class="action-btn primary" @click="playAgain">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 12"/>
            <path d="M3 3v9h9"/>
          </svg>
          再来一局
        </button>
        <button class="action-btn secondary" @click="restart">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/>
            <circle cx="9" cy="7" r="4"/>
            <path d="M23 21v-2a4 4 0 0 0-3-3.87"/>
            <path d="M16 3.13a4 4 0 0 1 0 7.75"/>
          </svg>
          换人玩
        </button>
      </div>
    </main>
  </div>
</template>

<style scoped>
/* ============ 基础布局 ============ */
.reaction-game {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: #0a0a14;
  background-image:
    radial-gradient(ellipse at 50% 0%, rgba(255, 107, 107, 0.08) 0%, transparent 60%),
    radial-gradient(ellipse at 80% 100%, rgba(255, 167, 38, 0.06) 0%, transparent 50%);
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
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
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
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}

.icon-btn:active {
  transform: scale(0.92);
  background: rgba(255, 255, 255, 0.12);
}

.title {
  flex: 1;
  text-align: center;
  font-size: 1.1rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  background: linear-gradient(135deg, #ff6b6b 0%, #ffa726 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header-spacer { width: 42px; }

/* ============ 主区域 ============ */
.main-area {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 20px 16px 40px;
  padding-bottom: max(40px, env(safe-area-inset-bottom));
  -webkit-overflow-scrolling: touch;
}

/* ============ 设置阶段 ============ */
.setup-intro {
  text-align: center;
  margin-bottom: 28px;
}

.intro-icon {
  margin-bottom: 12px;
}

.intro-title {
  font-size: 1.4rem;
  font-weight: 800;
  color: #fff;
  margin-bottom: 6px;
}

.intro-desc {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
}

.section-block {
  margin-bottom: 24px;
}

.section-label {
  font-size: 0.8rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.4);
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 12px;
}

.count-pills {
  display: flex;
  justify-content: center;
  gap: 8px;
  flex-wrap: wrap;
}

/* Staggered pill entrance */
@keyframes pillStaggerIn {
  0% {
    opacity: 0;
    transform: translateY(16px) scale(0.85);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.pill-stagger {
  animation: pillStaggerIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.pill {
  width: 46px;
  height: 46px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.05);
  border: 1.5px solid rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.5);
  font-size: 1.1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  -webkit-tap-highlight-color: transparent;
}

.pill:active { transform: scale(0.9); }

.pill.active {
  background: linear-gradient(135deg, #ff6b6b 0%, #ffa726 100%);
  border-color: transparent;
  color: #fff;
  box-shadow: 0 4px 16px rgba(255, 107, 107, 0.35);
  transform: scale(1.05);
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

.start-btn {
  width: 100%;
  height: 54px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: linear-gradient(135deg, #ff6b6b, #ffa726);
  border: none;
  border-radius: 16px;
  color: #fff;
  font-size: 1.05rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(255, 107, 107, 0.35);
  transition: all 0.2s cubic-bezier(0.22, 1, 0.36, 1);
  -webkit-tap-highlight-color: transparent;
  margin-bottom: 24px;
}

.start-btn:active {
  transform: scale(0.97);
  box-shadow: 0 2px 10px rgba(255, 107, 107, 0.4);
}

/* 规则卡片 */
.rules-card {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 18px;
  padding: 18px;
}

.rules-title {
  font-size: 0.85rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.5);
  margin-bottom: 14px;
}

.rule-item {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.55);
  margin-bottom: 10px;
}

.rule-item:last-child { margin-bottom: 0; }

.rule-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  flex-shrink: 0;
}

.waiting-dot { background: linear-gradient(135deg, #ff6b6b, #ffa726); }
.go-dot { background: linear-gradient(135deg, #00e676, #69f0ae); }
.foul-dot { background: #ff5252; }
.slow-dot { background: #ffa726; }

/* ============ 游戏全屏 ============ */
.game-fullscreen {
  flex: 1;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

/* --- idle 状态 --- */
.state-idle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 20px;
}

.idle-card {
  background: rgba(255, 255, 255, 0.04);
  border: 1.5px solid rgba(255, 255, 255, 0.08);
  border-radius: 24px;
  padding: 32px 24px;
  text-align: center;
  width: 100%;
  max-width: 340px;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}

.idle-round {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.4);
  font-weight: 500;
  letter-spacing: 1px;
  margin-bottom: 16px;
}

.idle-player {
  font-size: 2rem;
  font-weight: 800;
  background: linear-gradient(135deg, #ff6b6b, #ffa726);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 8px;
}

.idle-hint {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.45);
  margin-bottom: 24px;
}

.idle-go-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 14px 36px;
  background: linear-gradient(135deg, #ff6b6b, #ffa726);
  border: none;
  border-radius: 16px;
  color: #fff;
  font-size: 1.05rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(255, 107, 107, 0.35);
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}

.idle-go-btn:active {
  transform: scale(0.95);
}

.progress-dots {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 24px;
  padding-top: 20px;
  border-top: 1px solid rgba(255, 255, 255, 0.06);
}

.p-dot {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 14px;
  background: rgba(255, 255, 255, 0.04);
  border-radius: 10px;
}

.p-dot.foul {
  background: rgba(255, 82, 82, 0.08);
}

.p-dot-name {
  font-size: 0.85rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.7);
}

.p-dot-time {
  font-size: 0.85rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.5);
}

.p-dot.foul .p-dot-time {
  color: #ff5252;
}

/* --- waiting 状态 with enhanced breathing --- */
.state-waiting {
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, #bf360c 0%, #e65100 30%, #ff6d00 60%, #ff8f00 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  animation: waitingBreathe 2s cubic-bezier(0.22, 1, 0.36, 1) infinite;
}

@keyframes waitingBreathe {
  0%, 100% {
    filter: brightness(1);
    transform: scale(1);
  }
  50% {
    filter: brightness(1.12);
    transform: scale(1.005);
  }
}

.state-waiting::after {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 0, 0.15) 100%);
  animation: breatheOverlay 2s cubic-bezier(0.22, 1, 0.36, 1) infinite;
  pointer-events: none;
}

@keyframes breatheOverlay {
  0%, 100% { opacity: 0.3; }
  50% { opacity: 0; }
}

.waiting-content {
  text-align: center;
  z-index: 1;
}

.waiting-dots {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.w-dot {
  width: 14px;
  height: 14px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 50%;
  animation: dotBounce 1.2s cubic-bezier(0.22, 1, 0.36, 1) infinite;
}

@keyframes dotBounce {
  0%, 80%, 100% { transform: scale(0.6); opacity: 0.4; }
  40% { transform: scale(1.2); opacity: 1; }
}

.waiting-text {
  font-size: 3rem;
  font-weight: 900;
  color: #fff;
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  margin-bottom: 8px;
}

.waiting-sub {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 6px;
}

.waiting-warn {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
}

/* --- go 状态 with stronger flash --- */
.state-go {
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, #00c853 0%, #00e676 30%, #69f0ae 60%, #b9f6ca 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  animation: goFlashStrong 0.35s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes goFlashStrong {
  0% {
    filter: brightness(3);
    transform: scale(1.03);
  }
  30% {
    filter: brightness(1.8);
  }
  60% {
    filter: brightness(1.2);
    transform: scale(1.01);
  }
  100% {
    filter: brightness(1);
    transform: scale(1);
  }
}

.state-go::before {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(circle at center, rgba(255, 255, 255, 0.5) 0%, transparent 60%);
  animation: goRipple 0.6s cubic-bezier(0.22, 1, 0.36, 1) forwards;
  pointer-events: none;
}

@keyframes goRipple {
  0% { opacity: 1; transform: scale(0.3); }
  100% { opacity: 0; transform: scale(2); }
}

.go-content {
  text-align: center;
  animation: goPopIn 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  z-index: 1;
}

@keyframes goPopIn {
  0% { transform: scale(0.5); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}

.go-icon {
  margin-bottom: 12px;
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.2));
}

.go-text {
  font-size: 4rem;
  font-weight: 900;
  color: #fff;
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);
  margin-bottom: 8px;
}

.go-sub {
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.85);
  font-weight: 600;
}

/* --- foul 状态 with more violent shake --- */
.state-foul {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.92);
  display: flex;
  align-items: center;
  justify-content: center;
  animation: foulShakeViolent 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes foulShakeViolent {
  0% { transform: translateX(0) rotate(0); background: rgba(255, 50, 50, 0.3); }
  8% { transform: translateX(-16px) rotate(-1deg); }
  16% { transform: translateX(16px) rotate(1deg); background: rgba(255, 50, 50, 0.15); }
  24% { transform: translateX(-14px) rotate(-0.8deg); }
  32% { transform: translateX(14px) rotate(0.8deg); }
  40% { transform: translateX(-10px) rotate(-0.5deg); background: rgba(255, 50, 50, 0.08); }
  50% { transform: translateX(10px) rotate(0.5deg); }
  60% { transform: translateX(-6px) rotate(-0.3deg); }
  70% { transform: translateX(6px) rotate(0.3deg); }
  80% { transform: translateX(-3px) rotate(0); background: rgba(0, 0, 0, 0.92); }
  90% { transform: translateX(2px); }
  100% { transform: translateX(0) rotate(0); background: rgba(0, 0, 0, 0.92); }
}

.foul-content {
  text-align: center;
  animation: foulPopIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes foulPopIn {
  0% { transform: scale(0.6); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}

.foul-icon {
  margin-bottom: 16px;
  color: #ff5252;
}

.foul-icon svg {
  stroke: #ff5252;
}

.foul-title {
  font-size: 3rem;
  font-weight: 900;
  color: #ff5252;
  margin-bottom: 8px;
}

.foul-player {
  font-size: 1.3rem;
  color: rgba(255, 255, 255, 0.8);
  font-weight: 600;
  margin-bottom: 16px;
}

.foul-penalty {
  font-size: 1.1rem;
  color: #ffa726;
  font-weight: 700;
  margin-bottom: 32px;
  padding: 10px 24px;
  background: rgba(255, 167, 38, 0.1);
  border: 1px solid rgba(255, 167, 38, 0.2);
  border-radius: 12px;
  display: inline-block;
}

.foul-next-btn {
  display: block;
  width: 200px;
  margin: 0 auto;
  padding: 14px 24px;
  background: rgba(255, 255, 255, 0.1);
  border: 1.5px solid rgba(255, 255, 255, 0.15);
  border-radius: 14px;
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}

.foul-next-btn:active {
  transform: scale(0.95);
  background: rgba(255, 255, 255, 0.15);
}

/* --- done 状态 --- */
.state-done {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.88);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.done-content {
  text-align: center;
  width: 100%;
  max-width: 340px;
  animation: donePopIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes donePopIn {
  0% { transform: scale(0.7); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}

.done-player {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.5);
  font-weight: 500;
  margin-bottom: 12px;
}

.done-time {
  font-size: 5rem;
  font-weight: 900;
  line-height: 1;
  margin-bottom: 4px;
  font-variant-numeric: tabular-nums;
}

.done-unit {
  font-size: 1.8rem;
  font-weight: 600;
  opacity: 0.7;
}

.done-grade {
  font-size: 1.2rem;
  font-weight: 700;
  margin-bottom: 28px;
}

.done-bar-wrap {
  margin-bottom: 32px;
}

.done-bar-bg {
  width: 100%;
  height: 8px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 8px;
}

.done-bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

.done-bar-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.65rem;
  color: rgba(255, 255, 255, 0.25);
}

.done-next-btn {
  display: block;
  width: 100%;
  max-width: 240px;
  margin: 0 auto;
  padding: 14px 24px;
  background: linear-gradient(135deg, #ff6b6b, #ffa726);
  border: none;
  border-radius: 14px;
  color: #fff;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(255, 107, 107, 0.3);
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}

.done-next-btn:active {
  transform: scale(0.95);
}

/* ============ 结果阶段 ============ */
.result-header {
  text-align: center;
  margin-bottom: 24px;
}

.result-trophy {
  margin-bottom: 10px;
}

.result-title {
  font-size: 1.5rem;
  font-weight: 800;
  background: linear-gradient(135deg, #ffd700, #ffa726);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* 排行榜 with staggered entrance */
.leaderboard {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 24px;
}

@keyframes lbRowSlideIn {
  0% {
    opacity: 0;
    transform: translateX(-20px);
  }
  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

.lb-row-stagger {
  animation: lbRowSlideIn 0.45s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.lb-row {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 16px;
  background: rgba(255, 255, 255, 0.04);
  border: 1.5px solid rgba(255, 255, 255, 0.06);
  border-radius: 18px;
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.lb-row.rank-1 {
  background: linear-gradient(135deg, rgba(255, 215, 0, 0.08), rgba(255, 167, 38, 0.05));
  border-color: rgba(255, 215, 0, 0.2);
}

.lb-row.is-loser {
  background: linear-gradient(135deg, rgba(255, 82, 82, 0.1), rgba(255, 107, 107, 0.05));
  border-color: rgba(255, 82, 82, 0.25);
}

.lb-row.is-foul {
  background: rgba(255, 82, 82, 0.06);
  border-color: rgba(255, 82, 82, 0.15);
}

.lb-rank {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.rank-num {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.06);
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 800;
  color: rgba(255, 255, 255, 0.6);
}

.lb-row.rank-1 .rank-num {
  background: linear-gradient(135deg, #ffd700, #ffa726);
  color: #000;
}

.lb-info {
  flex: 1;
}

.lb-name {
  font-size: 1rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 2px;
}

.lb-time-text {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.4);
  font-variant-numeric: tabular-nums;
}

.lb-foul-tag {
  font-size: 0.8rem;
  color: #ff5252;
  font-weight: 600;
}

.lb-badge-area {
  display: flex;
  gap: 6px;
  flex-shrink: 0;
}

.lb-badge {
  padding: 5px 12px;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: 700;
}

.lb-badge.fastest {
  background: rgba(255, 215, 0, 0.15);
  color: #ffd700;
  border: 1px solid rgba(255, 215, 0, 0.25);
}

.lb-badge.drink {
  background: rgba(255, 82, 82, 0.15);
  color: #ff5252;
  border: 1px solid rgba(255, 82, 82, 0.25);
  animation: drinkPulse 1.5s cubic-bezier(0.22, 1, 0.36, 1) infinite;
}

@keyframes drinkPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.08); }
}

/* 统计卡片 with pop-in */
.stats-card {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-bottom: 24px;
}

@keyframes statCardPop {
  0% {
    opacity: 0;
    transform: translateY(12px) scale(0.92);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.stat-pop {
  animation: statCardPop 0.4s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes statNumPop {
  0% {
    opacity: 0;
    transform: scale(0.5);
  }
  60% {
    transform: scale(1.1);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

.stat-num-pop {
  animation: statNumPop 0.5s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.stat-item {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 14px;
  padding: 14px;
  text-align: center;
}

.stat-label {
  display: block;
  font-size: 0.72rem;
  color: rgba(255, 255, 255, 0.35);
  font-weight: 500;
  margin-bottom: 6px;
}

.stat-value {
  display: block;
  font-size: 1.2rem;
  font-weight: 800;
  color: rgba(255, 255, 255, 0.8);
  font-variant-numeric: tabular-nums;
}

.stat-value.foul-val {
  color: #ff5252;
}

/* 操作按钮 */
.result-actions {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.action-btn {
  width: 100%;
  height: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  border: none;
  border-radius: 16px;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.22, 1, 0.36, 1);
  -webkit-tap-highlight-color: transparent;
}

.action-btn:active { transform: scale(0.97); }

.action-btn.primary {
  background: linear-gradient(135deg, #ff6b6b, #ffa726);
  color: #fff;
  box-shadow: 0 4px 20px rgba(255, 107, 107, 0.3);
}

.action-btn.secondary {
  background: rgba(255, 255, 255, 0.06);
  border: 1.5px solid rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.8);
}

/* ============ 滚动条美化 ============ */
.main-area::-webkit-scrollbar {
  width: 3px;
}

.main-area::-webkit-scrollbar-track {
  background: transparent;
}

.main-area::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 3px;
}

/* ============ 响应式 ============ */
@media (max-height: 700px) {
  .intro-icon svg { width: 40px; height: 40px; }
  .intro-title { font-size: 1.2rem; }
  .idle-player { font-size: 1.6rem; }
  .waiting-text { font-size: 2.5rem; }
  .go-text { font-size: 3rem; }
  .done-time { font-size: 3.5rem; }
  .foul-title { font-size: 2.5rem; }
}
</style>
