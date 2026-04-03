<script setup>
import { ref, computed } from 'vue'

defineEmits(['goBack'])

const triggerHaptic = () => {
  if ('vibrate' in navigator) {
    navigator.vibrate(25)
  }
}

// ===== 游戏状态 =====
const gamePhase = ref('setup') // 'setup' | 'playing' | 'reveal'
const playerCount = ref(4)
const playerNames = ref([])
const kingIndex = ref(-1)
const executorIndex = ref(-1)
const currentCommand = ref('')
const isFlipping = ref(false)
const isCommandRevealed = ref(false)
const roundCount = ref(0)
const showHistory = ref(false)
const history = ref([])

// ===== 预置命令列表 =====
const commands = [
  '喝一杯',
  '表演一个才艺',
  '说出自己最尴尬的事',
  '模仿一种动物',
  '对左边的人说一句情话',
  '做10个俯卧撑',
  '唱一首歌的副歌部分',
  '讲一个笑话，不好笑就喝酒',
  '和右边的人猜拳，输的喝一杯',
  '模仿在场任意一人的说话方式',
  '大声喊出"我最帅/美"',
  '给任意一人一个拥抱',
  '用方言说一句话',
  '做一个鬼脸并保持10秒',
  '说出三个在场某人的优点',
  '闭眼原地转三圈',
  '学一种动物叫声持续10秒',
  '说出自己手机最后一条消息',
  '向在场任意一人鞠躬道歉',
  '和最近的人碰杯喝一口',
  '用一句话形容自己的长相',
  '做一段广播体操',
  '把手机交给旁边的人发一条朋友圈',
  '模仿一个明星说话',
  '对着手机前置摄像头自拍并展示',
  '说出在场某人的手机壁纸是什么',
  '连续说三个"我爱你"给不同的人',
  '表演一段慢动作喝水',
  '用歌词回答接下来别人问你的三个问题',
  '和对面的人深情对视10秒',
  '即兴rap一段自我介绍',
  '用左手给右边的人喂一口吃的',
  '模仿婴儿哭三秒然后大笑三秒',
  '说出你手机里最常联系的人是谁',
  '表演一个电影经典片段',
]

// ===== 初始化玩家名字 =====
const initPlayerNames = () => {
  playerNames.value = Array.from({ length: playerCount.value }, (_, i) => `玩家${i + 1}`)
}
initPlayerNames()

// ===== 计算属性 =====
const kingName = computed(() => {
  if (kingIndex.value < 0) return ''
  return playerNames.value[kingIndex.value] || `玩家${kingIndex.value + 1}`
})

const executorName = computed(() => {
  if (executorIndex.value < 0) return ''
  return playerNames.value[executorIndex.value] || `玩家${executorIndex.value + 1}`
})

const getInitial = (name) => {
  return name ? name.charAt(0) : '?'
}

// ===== 人数调整 =====
const adjustCount = (delta) => {
  triggerHaptic()
  const newCount = playerCount.value + delta
  if (newCount >= 3 && newCount <= 10) {
    playerCount.value = newCount
    // 扩展或缩减名字数组
    if (delta > 0) {
      playerNames.value.push(`玩家${newCount}`)
    } else {
      playerNames.value.pop()
    }
  }
}

// ===== 开始游戏 =====
const startGame = () => {
  triggerHaptic()
  // 确保每个名字都不为空
  playerNames.value = playerNames.value.map((name, i) => name.trim() || `玩家${i + 1}`)
  gamePhase.value = 'playing'
  roundCount.value = 0
  history.value = []
  nextRound()
}

// ===== 下一轮 =====
const nextRound = () => {
  triggerHaptic()
  isCommandRevealed.value = false
  isFlipping.value = false
  roundCount.value++

  // 随机选国王
  kingIndex.value = Math.floor(Math.random() * playerCount.value)

  // 随机选执行者（不能是国王）
  let exec
  do {
    exec = Math.floor(Math.random() * playerCount.value)
  } while (exec === kingIndex.value)
  executorIndex.value = exec

  // 随机选命令
  currentCommand.value = commands[Math.floor(Math.random() * commands.length)]

  gamePhase.value = 'playing'
}

// ===== 翻牌揭晓命令 =====
const revealCommand = () => {
  if (isCommandRevealed.value || isFlipping.value) return
  triggerHaptic()
  isFlipping.value = true
  setTimeout(() => {
    isCommandRevealed.value = true
    setTimeout(() => {
      isFlipping.value = false
    }, 400)
  }, 300)
}

// ===== 完成本轮 =====
const finishRound = () => {
  triggerHaptic()
  history.value.unshift({
    round: roundCount.value,
    king: kingName.value,
    executor: executorName.value,
    command: currentCommand.value,
  })
  gamePhase.value = 'reveal'
}

// ===== 重置游戏 =====
const resetGame = () => {
  triggerHaptic()
  gamePhase.value = 'setup'
  kingIndex.value = -1
  executorIndex.value = -1
  currentCommand.value = ''
  isCommandRevealed.value = false
  isFlipping.value = false
  roundCount.value = 0
  history.value = []
}
</script>

<template>
  <div class="king-game">
    <!-- 背景金色粒子装饰 -->
    <div class="bg-decoration">
      <div class="bg-orb bg-orb-1"></div>
      <div class="bg-orb bg-orb-2"></div>
      <div class="bg-orb bg-orb-3"></div>
      <div class="bg-sparkle bg-sparkle-1"></div>
      <div class="bg-sparkle bg-sparkle-2"></div>
      <div class="bg-sparkle bg-sparkle-3"></div>
      <div class="bg-sparkle bg-sparkle-4"></div>
    </div>

    <!-- 顶部标题栏 -->
    <header class="mobile-header">
      <button class="back-btn" @click="$emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title">👑 国王游戏</h1>
      <button v-if="gamePhase !== 'setup'" class="history-btn" @click="showHistory = !showHistory">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"/>
          <polyline points="12 6 12 12 16 14"/>
        </svg>
      </button>
      <div v-else class="header-spacer"></div>
    </header>

    <!-- ===== 设置阶段 ===== -->
    <main v-if="gamePhase === 'setup'" class="game-area">
      <div class="setup-container">
        <div class="setup-intro">
          <div class="intro-crown">👑</div>
          <div class="intro-title">国王游戏</div>
          <div class="intro-desc">每轮随机选出国王，国王下达命令！</div>
        </div>

        <!-- 人数选择 -->
        <div class="setting-section">
          <div class="setting-label">参与人数</div>
          <div class="counter-row">
            <button class="counter-btn" :disabled="playerCount <= 3" @click="adjustCount(-1)">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <line x1="5" y1="12" x2="19" y2="12"/>
              </svg>
            </button>
            <span class="counter-value">{{ playerCount }}</span>
            <button class="counter-btn" :disabled="playerCount >= 10" @click="adjustCount(1)">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <line x1="12" y1="5" x2="12" y2="19"/>
                <line x1="5" y1="12" x2="19" y2="12"/>
              </svg>
            </button>
          </div>
        </div>

        <!-- 玩家名字列表 -->
        <div class="setting-section">
          <div class="setting-label">玩家名字</div>
          <div class="player-inputs">
            <div
              v-for="(name, index) in playerNames"
              :key="index"
              class="player-input-row"
              :style="{ animationDelay: (index * 0.06) + 's' }"
            >
              <div class="player-avatar-small" :style="{ background: `hsl(${index * 37}, 65%, 55%)` }">
                {{ getInitial(name) }}
              </div>
              <input
                v-model="playerNames[index]"
                type="text"
                class="player-name-input"
                :placeholder="`玩家${index + 1}`"
                maxlength="6"
              />
            </div>
          </div>
        </div>

        <!-- 开始按钮 -->
        <button class="start-btn" @click="startGame">
          <span class="start-crown">👑</span>
          开始游戏
        </button>
      </div>
    </main>

    <!-- ===== 游戏进行中 ===== -->
    <main v-else class="game-area">
      <div class="playing-container">
        <!-- 轮次信息 -->
        <div class="round-badge">第 {{ roundCount }} 轮</div>

        <!-- 玩家环 -->
        <div class="players-ring">
          <div
            v-for="(name, index) in playerNames"
            :key="index"
            class="player-chip"
            :class="{
              'is-king': index === kingIndex,
              'is-executor': index === executorIndex && isCommandRevealed,
            }"
            :style="{ animationDelay: (index * 0.06) + 's' }"
          >
            <div class="chip-avatar" :style="{ background: index === kingIndex ? 'linear-gradient(135deg, #ffd700, #ffaa00)' : (index === executorIndex && isCommandRevealed ? 'linear-gradient(135deg, #ff4757, #ff6b81)' : `hsl(${index * 37}, 65%, 55%)`) }">
              <span v-if="index === kingIndex" class="crown-icon">👑</span>
              <span v-else>{{ getInitial(name) }}</span>
            </div>
            <div class="chip-name" :class="{ 'king-name': index === kingIndex, 'executor-name': index === executorIndex && isCommandRevealed }">
              {{ name }}
            </div>
            <div v-if="index === kingIndex" class="chip-role king-role">国王</div>
            <div v-else-if="index === executorIndex && isCommandRevealed" class="chip-role executor-role">执行者</div>
          </div>
        </div>

        <!-- 国王宣告 -->
        <div class="king-announce">
          <div class="announce-glow"></div>
          <div class="announce-shimmer"></div>
          <div class="announce-text">
            <span class="king-highlight">{{ kingName }}</span> 是本轮国王！
          </div>
        </div>

        <!-- 命令卡片 -->
        <div class="command-section">
          <div class="command-label">国王的命令</div>
          <div
            class="flip-container"
            :class="{ flipping: isFlipping }"
            @click="revealCommand"
          >
            <div class="flip-card" :class="{ flipped: isCommandRevealed }">
              <!-- 正面：未揭晓 -->
              <div class="flip-front">
                <div class="card-pattern"></div>
                <div class="front-crown">👑</div>
                <div class="front-text">点击揭晓命令</div>
              </div>
              <!-- 背面：命令内容 -->
              <div class="flip-back">
                <div class="command-content">
                  <div class="command-text">{{ currentCommand }}</div>
                  <div v-if="isCommandRevealed" class="executor-announce">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                      <path d="M5 12h14M12 5l7 7-7 7"/>
                    </svg>
                    <span class="executor-highlight">{{ executorName }}</span> 来执行！
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 提示文字 -->
        <div v-if="isCommandRevealed && gamePhase === 'playing'" class="penalty-hint">
          不执行？那就喝一杯吧！🍺
        </div>

        <!-- 操作按钮 -->
        <div class="action-row">
          <button class="action-btn secondary" @click="resetGame">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M19 12H5M12 19l-7-7 7-7"/>
            </svg>
            重新开始
          </button>
          <button
            class="action-btn primary"
            :disabled="!isCommandRevealed"
            @click="finishRound(); nextRound()"
          >
            下一轮
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M5 12h14M12 5l7 7-7 7"/>
            </svg>
          </button>
        </div>
      </div>
    </main>

    <!-- ===== 历史记录弹窗 ===== -->
    <Transition name="slide-up">
      <div v-if="showHistory" class="history-overlay" @click.self="showHistory = false">
        <div class="history-panel">
          <div class="history-header">
            <h2 class="history-title">历史记录</h2>
            <button class="close-btn" @click="showHistory = false">
              <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <line x1="18" y1="6" x2="6" y2="18"/>
                <line x1="6" y1="6" x2="18" y2="18"/>
              </svg>
            </button>
          </div>
          <div v-if="history.length === 0" class="history-empty">
            暂无记录
          </div>
          <div v-else class="history-list">
            <div v-for="item in history" :key="item.round" class="history-item">
              <div class="history-round">第{{ item.round }}轮</div>
              <div class="history-detail">
                <span class="h-king">👑 {{ item.king }}</span>
                命令
                <span class="h-executor">{{ item.executor }}</span>：
                <span class="h-command">{{ item.command }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
/* ===== 全局容器 ===== */
.king-game {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: #0a0a14;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding-bottom: env(safe-area-inset-bottom);
}

/* ===== 背景金色粒子装饰 ===== */
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
  background: radial-gradient(circle, rgba(255, 215, 0, 0.2), transparent 70%);
  top: 8%; left: -5%;
  animation-delay: 0s;
}
.bg-orb-2 {
  width: 150px; height: 150px;
  background: radial-gradient(circle, rgba(255, 170, 0, 0.15), transparent 70%);
  top: 55%; right: -8%;
  animation-delay: -5s;
}
.bg-orb-3 {
  width: 120px; height: 120px;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.12), transparent 70%);
  bottom: 12%; left: 25%;
  animation-delay: -9s;
}

@keyframes orbFloat {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(12px, -18px) scale(1.05); }
  66% { transform: translate(-8px, 12px) scale(0.95); }
}

.bg-sparkle {
  position: absolute;
  width: 3px; height: 3px;
  background: rgba(255, 215, 0, 0.5);
  border-radius: 50%;
  animation: sparkleFloat 6s ease-in-out infinite;
}

.bg-sparkle-1 { top: 15%; left: 20%; animation-delay: 0s; }
.bg-sparkle-2 { top: 35%; right: 15%; animation-delay: -1.5s; }
.bg-sparkle-3 { bottom: 30%; left: 10%; animation-delay: -3s; }
.bg-sparkle-4 { bottom: 20%; right: 25%; animation-delay: -4.5s; }

@keyframes sparkleFloat {
  0%, 100% { opacity: 0; transform: translateY(0) scale(0.5); }
  25% { opacity: 0.7; transform: translateY(-15px) scale(1); }
  50% { opacity: 0.3; transform: translateY(-30px) scale(0.8); }
  75% { opacity: 0.6; transform: translateY(-45px) scale(1.1); }
}

/* ===== 顶部标题栏 ===== */
.mobile-header {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  padding-top: max(12px, env(safe-area-inset-top));
  background: rgba(10, 10, 20, 0.75);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(255, 215, 0, 0.08);
  z-index: 10;
  position: relative;
}

.back-btn {
  width: 44px; height: 44px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  color: rgba(255, 255, 255, 0.85);
  cursor: pointer;
  touch-action: manipulation;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.back-btn:active {
  background: rgba(255, 255, 255, 0.12);
  transform: scale(0.93);
}

.header-title {
  flex: 1;
  text-align: center;
  font-size: 1.1rem;
  font-weight: 700;
  background: linear-gradient(135deg, #ffd700 0%, #ffaa00 50%, #ffd700 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin: 0 8px;
  letter-spacing: 0.5px;
}

.header-spacer { width: 44px; }

.history-btn {
  width: 44px; height: 44px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 215, 0, 0.08);
  border: 1px solid rgba(255, 215, 0, 0.15);
  border-radius: 14px;
  color: #ffd700;
  cursor: pointer;
  touch-action: manipulation;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.history-btn:active {
  background: rgba(255, 215, 0, 0.18);
  transform: scale(0.93);
}

/* ===== 主游戏区域 ===== */
.game-area {
  flex: 1;
  overflow-y: auto;
  padding: 20px 16px;
  padding-bottom: max(24px, env(safe-area-inset-bottom));
  -webkit-overflow-scrolling: touch;
  position: relative;
  z-index: 1;
}

/* ===== 设置阶段 ===== */
.setup-container {
  max-width: 400px;
  margin: 0 auto;
}

.setup-intro {
  text-align: center;
  margin-bottom: 36px;
  animation: introFadeIn 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes introFadeIn {
  from { opacity: 0; transform: translateY(16px); }
  to { opacity: 1; transform: translateY(0); }
}

.intro-crown {
  font-size: 4rem;
  margin-bottom: 12px;
  animation: floatCrown 3s ease-in-out infinite;
}
@keyframes floatCrown {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  25% { transform: translateY(-6px) rotate(-3deg); }
  75% { transform: translateY(-3px) rotate(3deg); }
}
.intro-title {
  font-size: 1.5rem;
  font-weight: 700;
  color: #ffd700;
  margin-bottom: 8px;
  letter-spacing: 1px;
}
.intro-desc {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.4);
  line-height: 1.5;
}

.setting-section {
  margin-bottom: 24px;
  animation: sectionSlideIn 0.5s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.setting-section:nth-child(2) { animation-delay: 0.1s; }
.setting-section:nth-child(3) { animation-delay: 0.2s; }

@keyframes sectionSlideIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

.setting-label {
  font-size: 0.88rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.55);
  margin-bottom: 12px;
  padding-left: 2px;
}

/* 人数计数器 */
.counter-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 24px;
}
.counter-btn {
  width: 48px; height: 48px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 215, 0, 0.06);
  border: 1.5px solid rgba(255, 215, 0, 0.15);
  border-radius: 16px;
  color: #ffd700;
  cursor: pointer;
  touch-action: manipulation;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.counter-btn:active:not(:disabled) {
  background: rgba(255, 215, 0, 0.2);
  transform: scale(0.93);
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.2);
  border-color: rgba(255, 215, 0, 0.4);
}
.counter-btn:disabled {
  opacity: 0.25;
  cursor: not-allowed;
}
.counter-value {
  font-size: 2.2rem;
  font-weight: 700;
  color: #ffd700;
  min-width: 60px;
  text-align: center;
}

/* 玩家名字输入 */
.player-inputs {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.player-input-row {
  display: flex;
  align-items: center;
  gap: 12px;
  animation: playerStagger 0.4s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes playerStagger {
  from { opacity: 0; transform: translateX(-14px); }
  to { opacity: 1; transform: translateX(0); }
}

.player-avatar-small {
  width: 38px; height: 38px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 0.85rem;
  font-weight: 700;
  color: #fff;
  flex-shrink: 0;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.player-name-input {
  flex: 1;
  height: 44px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  padding: 0 14px;
  color: #fff;
  font-size: 0.95rem;
  outline: none;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.player-name-input:focus {
  border-color: rgba(255, 215, 0, 0.4);
  box-shadow: 0 0 0 3px rgba(255, 215, 0, 0.08);
}
.player-name-input::placeholder {
  color: rgba(255, 255, 255, 0.2);
}

/* 开始按钮 */
.start-btn {
  width: 100%;
  height: 56px;
  margin-top: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: linear-gradient(135deg, #ffd700 0%, #ffaa00 100%);
  border: none;
  border-radius: 18px;
  font-size: 1.1rem;
  font-weight: 700;
  color: #0a0a14;
  cursor: pointer;
  touch-action: manipulation;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  box-shadow: 0 4px 24px rgba(255, 215, 0, 0.25);
  position: relative;
  overflow: hidden;
}

.start-btn::after {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  animation: shimmerGold 2.5s ease-in-out infinite;
}

@keyframes shimmerGold {
  0% { left: -100%; }
  50%, 100% { left: 100%; }
}

.start-btn:active {
  transform: scale(0.96);
  box-shadow: 0 2px 12px rgba(255, 215, 0, 0.2);
}
.start-crown {
  font-size: 1.3rem;
}

/* ===== 游戏进行阶段 ===== */
.playing-container {
  max-width: 400px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.round-badge {
  display: inline-block;
  padding: 6px 18px;
  background: rgba(255, 215, 0, 0.08);
  border: 1px solid rgba(255, 215, 0, 0.15);
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 600;
  color: #ffd700;
  margin-bottom: 20px;
  letter-spacing: 1px;
  animation: badgeFadeIn 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes badgeFadeIn {
  from { opacity: 0; transform: scale(0.8); }
  to { opacity: 1; transform: scale(1); }
}

/* 玩家环 */
.players-ring {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 12px;
  margin-bottom: 24px;
  width: 100%;
}

.player-chip {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  min-width: 60px;
  transition: transform 0.3s cubic-bezier(0.22, 1, 0.36, 1);
  animation: chipStagger 0.5s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes chipStagger {
  from { opacity: 0; transform: translateY(16px) scale(0.8); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}

.player-chip.is-king {
  transform: scale(1.1);
}
.player-chip.is-executor {
  animation: chipStagger 0.5s cubic-bezier(0.22, 1, 0.36, 1) both, executorPulse 1.5s ease-in-out infinite;
}
@keyframes executorPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.08); }
}

.chip-avatar {
  width: 48px; height: 48px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 0.95rem;
  font-weight: 700;
  color: #fff;
  position: relative;
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}
.is-king .chip-avatar {
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.4), 0 0 40px rgba(255, 215, 0, 0.15);
  width: 54px; height: 54px;
  font-size: 1.4rem;
}
.is-executor .chip-avatar {
  box-shadow: 0 0 20px rgba(255, 71, 87, 0.4), 0 0 40px rgba(255, 71, 87, 0.15);
}

.crown-icon {
  font-size: 1.5rem;
}

.chip-name {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.5);
  max-width: 64px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  text-align: center;
}
.chip-name.king-name {
  color: #ffd700;
  font-weight: 600;
}
.chip-name.executor-name {
  color: #ff4757;
  font-weight: 600;
}

.chip-role {
  font-size: 0.65rem;
  padding: 2px 8px;
  border-radius: 8px;
  font-weight: 600;
}
.king-role {
  background: rgba(255, 215, 0, 0.15);
  color: #ffd700;
  border: 1px solid rgba(255, 215, 0, 0.2);
}
.executor-role {
  background: rgba(255, 71, 87, 0.15);
  color: #ff4757;
  border: 1px solid rgba(255, 71, 87, 0.2);
  animation: executorBadgePulse 1.5s ease-in-out infinite;
}
@keyframes executorBadgePulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(255, 71, 87, 0.3); }
  50% { box-shadow: 0 0 0 4px rgba(255, 71, 87, 0); }
}

/* 国王宣告 */
.king-announce {
  position: relative;
  width: 100%;
  padding: 18px 20px;
  background: rgba(255, 215, 0, 0.05);
  border: 1px solid rgba(255, 215, 0, 0.12);
  border-radius: 18px;
  text-align: center;
  margin-bottom: 24px;
  overflow: hidden;
  animation: announceIn 0.5s cubic-bezier(0.22, 1, 0.36, 1);
  box-shadow: 0 0 40px rgba(255, 215, 0, 0.06), inset 0 0 30px rgba(255, 215, 0, 0.02);
}

@keyframes announceIn {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}

.announce-glow {
  position: absolute;
  top: -50px; left: 50%;
  transform: translateX(-50%);
  width: 200px; height: 100px;
  background: radial-gradient(ellipse, rgba(255, 215, 0, 0.18), transparent 70%);
  pointer-events: none;
  animation: glowPulse 3s ease-in-out infinite;
}

@keyframes glowPulse {
  0%, 100% { opacity: 0.6; transform: translateX(-50%) scale(1); }
  50% { opacity: 1; transform: translateX(-50%) scale(1.15); }
}

.announce-shimmer {
  position: absolute;
  top: 0; left: -100%; right: 0; bottom: 0;
  width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 215, 0, 0.06), transparent);
  animation: announceShimmer 4s ease-in-out infinite;
  pointer-events: none;
}

@keyframes announceShimmer {
  0% { left: -100%; }
  50%, 100% { left: 100%; }
}

.announce-text {
  font-size: 1.05rem;
  color: rgba(255, 255, 255, 0.75);
  position: relative;
  z-index: 1;
}
.king-highlight {
  color: #ffd700;
  font-weight: 700;
  font-size: 1.15rem;
  text-shadow: 0 0 20px rgba(255, 215, 0, 0.3);
}

/* ===== 命令卡片翻转 ===== */
.command-section {
  width: 100%;
  margin-bottom: 16px;
}
.command-label {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.4);
  text-align: center;
  margin-bottom: 12px;
  letter-spacing: 1px;
}

.flip-container {
  width: 100%;
  min-height: 180px;
  perspective: 1200px;
  cursor: pointer;
  touch-action: manipulation;
}
.flip-container.flipping {
  animation: cardShake 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}
@keyframes cardShake {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(0.97); }
}

.flip-card {
  position: relative;
  width: 100%;
  min-height: 180px;
  transition: transform 0.7s cubic-bezier(0.22, 1, 0.36, 1);
  transform-style: preserve-3d;
}
.flip-card.flipped {
  transform: rotateY(180deg);
}

.flip-front,
.flip-back {
  position: absolute;
  top: 0; left: 0;
  width: 100%;
  min-height: 180px;
  border-radius: 18px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  overflow: hidden;
  transition: box-shadow 0.7s cubic-bezier(0.22, 1, 0.36, 1);
}

/* 正面 */
.flip-front {
  background: linear-gradient(135deg, rgba(255, 215, 0, 0.04), rgba(255, 170, 0, 0.02));
  border: 1.5px solid rgba(255, 215, 0, 0.12);
  z-index: 2;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

.flip-card.flipped .flip-front {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.card-pattern {
  position: absolute;
  inset: 0;
  opacity: 0.03;
  background-image:
    radial-gradient(circle at 25% 25%, #ffd700 1px, transparent 1px),
    radial-gradient(circle at 75% 75%, #ffd700 1px, transparent 1px);
  background-size: 24px 24px;
  pointer-events: none;
}
.front-crown {
  font-size: 3rem;
  margin-bottom: 10px;
  animation: bounceUp 1.8s cubic-bezier(0.36, 0, 0.66, 1) infinite;
  position: relative;
  z-index: 1;
}
@keyframes bounceUp {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}
.front-text {
  font-size: 1rem;
  color: rgba(255, 215, 0, 0.45);
  position: relative;
  z-index: 1;
  letter-spacing: 1px;
}

/* 背面 */
.flip-back {
  background: rgba(255, 215, 0, 0.03);
  border: 1.5px solid rgba(255, 215, 0, 0.25);
  box-shadow: 0 12px 40px rgba(255, 215, 0, 0.1), 0 0 60px rgba(255, 215, 0, 0.04), inset 0 0 30px rgba(255, 215, 0, 0.02);
  transform: rotateY(180deg);
  padding: 28px 24px;
}
.command-content {
  text-align: center;
  animation: contentFadeIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.15s both;
}
@keyframes contentFadeIn {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}
.command-text {
  font-size: 1.5rem;
  line-height: 1.6;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.95);
  margin-bottom: 18px;
}
.executor-announce {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.6);
  padding-top: 14px;
  border-top: 1px solid rgba(255, 255, 255, 0.06);
}
.executor-announce svg {
  color: #ff4757;
}
.executor-highlight {
  color: #ff4757;
  font-weight: 700;
  font-size: 1.1rem;
}

/* 惩罚提示 */
.penalty-hint {
  font-size: 0.88rem;
  color: rgba(255, 255, 255, 0.35);
  text-align: center;
  margin-bottom: 20px;
  letter-spacing: 0.5px;
  animation: fadeSlideUp 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes fadeSlideUp {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 操作按钮 */
.action-row {
  display: flex;
  gap: 12px;
  width: 100%;
  margin-top: 4px;
}
.action-btn {
  flex: 1;
  height: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  border: none;
  border-radius: 18px;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  touch-action: manipulation;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.action-btn:active:not(:disabled) {
  transform: scale(0.96);
}
.action-btn:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.action-btn.secondary {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.6);
}
.action-btn.secondary:active {
  background: rgba(255, 255, 255, 0.1);
}

.action-btn.primary {
  flex: 1.5;
  background: linear-gradient(135deg, #ffd700 0%, #ffaa00 100%);
  color: #0a0a14;
  box-shadow: 0 4px 20px rgba(255, 215, 0, 0.2);
  position: relative;
  overflow: hidden;
}

.action-btn.primary::after {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.25), transparent);
  animation: shimmerGold 2.5s ease-in-out infinite;
}

.action-btn.primary:active:not(:disabled) {
  box-shadow: 0 2px 10px rgba(255, 215, 0, 0.15);
}
.action-btn.primary:disabled {
  box-shadow: none;
}
.action-btn.primary:disabled::after {
  animation: none;
}

/* ===== 历史记录弹窗 ===== */
.history-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 100;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}
.history-panel {
  width: 100%;
  max-width: 500px;
  max-height: 70vh;
  background: #13132a;
  border-top-left-radius: 24px;
  border-top-right-radius: 24px;
  padding: 20px 16px;
  padding-bottom: max(20px, env(safe-area-inset-bottom));
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
.history-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}
.history-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.9);
  margin: 0;
}
.close-btn {
  width: 40px; height: 40px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  touch-action: manipulation;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}
.close-btn:active {
  background: rgba(255, 255, 255, 0.1);
}
.history-empty {
  text-align: center;
  color: rgba(255, 255, 255, 0.25);
  padding: 40px 0;
  font-size: 0.95rem;
}
.history-list {
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.history-item {
  padding: 14px 16px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 14px;
}
.history-round {
  font-size: 0.75rem;
  font-weight: 600;
  color: rgba(255, 215, 0, 0.6);
  margin-bottom: 6px;
}
.history-detail {
  font-size: 0.88rem;
  color: rgba(255, 255, 255, 0.55);
  line-height: 1.5;
}
.h-king {
  color: #ffd700;
  font-weight: 600;
}
.h-executor {
  color: #ff4757;
  font-weight: 600;
}
.h-command {
  color: rgba(255, 255, 255, 0.85);
  font-weight: 500;
}

/* ===== 历史弹窗动画 ===== */
.slide-up-enter-active {
  transition: all 0.35s cubic-bezier(0.22, 1, 0.36, 1);
}
.slide-up-leave-active {
  transition: all 0.25s ease-in;
}
.slide-up-enter-from .history-panel,
.slide-up-leave-to .history-panel {
  transform: translateY(100%);
}
.slide-up-enter-from,
.slide-up-leave-to {
  opacity: 0;
}

/* ===== 小屏适配 ===== */
@media (max-height: 700px) {
  .setup-intro { margin-bottom: 24px; }
  .intro-crown { font-size: 3rem; }
  .intro-title { font-size: 1.3rem; }
  .players-ring { gap: 8px; margin-bottom: 16px; }
  .chip-avatar { width: 42px; height: 42px; font-size: 0.85rem; }
  .is-king .chip-avatar { width: 48px; height: 48px; }
  .king-announce { padding: 12px 16px; margin-bottom: 16px; }
  .flip-container { min-height: 150px; }
  .flip-card { min-height: 150px; }
  .flip-front, .flip-back { min-height: 150px; }
  .command-text { font-size: 1.25rem; }
}

/* ===== 大屏适配 ===== */
@media (min-width: 500px) {
  .setup-container,
  .playing-container {
    max-width: 440px;
  }
}
</style>
