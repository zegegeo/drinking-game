<script setup>
import { ref } from 'vue'
import { truthQuestions, dareActions } from '../assets/data.js'
import { truthQuestionsSpicy, dareActionsSpicy } from '../assets/dataSpicy.js'

defineEmits(['goBack'])

const difficulty = ref(null)  // 'mild' | 'spicy'
const mode = ref(null)        // 'truth' | 'dare'
const currentContent = ref('')
const isRevealed = ref(false)
const isAnimating = ref(false)

const triggerHaptic = () => {
  if ('vibrate' in navigator) navigator.vibrate(25)
}

const getRandomItem = (array) => {
  if (!array || array.length === 0) return '题库为空，请先填充内容~'
  return array[Math.floor(Math.random() * array.length)]
}

const getTruthPool = () => difficulty.value === 'spicy' ? truthQuestionsSpicy : truthQuestions
const getDarePool = () => difficulty.value === 'spicy' ? dareActionsSpicy : dareActions

const selectDifficulty = (d) => {
  triggerHaptic()
  difficulty.value = d
}

const selectMode = (selectedMode) => {
  triggerHaptic()
  mode.value = selectedMode
  isRevealed.value = false
  isAnimating.value = true
  setTimeout(() => {
    currentContent.value = getRandomItem(selectedMode === 'truth' ? getTruthPool() : getDarePool())
    isAnimating.value = false
  }, 400)
}

const reveal = () => {
  if (!isRevealed.value && !isAnimating.value) {
    triggerHaptic()
    isRevealed.value = true
  }
}

const nextRound = () => {
  triggerHaptic()
  isRevealed.value = false
  isAnimating.value = true
  setTimeout(() => {
    currentContent.value = getRandomItem(mode.value === 'truth' ? getTruthPool() : getDarePool())
    isAnimating.value = false
  }, 300)
}

const backToModeSelect = () => {
  mode.value = null
  isRevealed.value = false
  currentContent.value = ''
}

const backToDiffSelect = () => {
  difficulty.value = null
  mode.value = null
  isRevealed.value = false
  currentContent.value = ''
}
</script>

<template>
  <div class="truth-dare-mobile">
    <div class="td-bg">
      <div class="td-mesh td-mesh-1"></div>
      <div class="td-mesh td-mesh-2"></div>
    </div>
    <div class="td-grain"></div>

    <header class="mobile-header">
      <button class="back-btn" @click="mode ? backToModeSelect() : difficulty ? backToDiffSelect() : $emit('goBack')">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <path d="M19 12H5M12 19l-7-7 7-7"/>
        </svg>
      </button>
      <h1 class="header-title">真心话大冒险</h1>
      <div class="header-spacer"></div>
    </header>

    <main class="game-area">
      <Transition name="page-fade" mode="out-in">

        <!-- 第一步：难度选择 -->
        <div v-if="!difficulty" key="diff" class="diff-select">
          <div class="diff-hero">
            <div class="diff-icon-wrap">
              <div class="diff-icon-glow"></div>
              <div class="diff-icon-inner">
                <svg width="44" height="44" viewBox="0 0 24 24" fill="none">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2z" fill="url(#hero-g)" opacity="0.15"/>
                  <path d="M8 14s1.5 2 4 2 4-2 4-2" stroke="url(#hero-g)" stroke-width="1.5" stroke-linecap="round"/>
                  <circle cx="9" cy="9.5" r="1.2" fill="url(#hero-g)"/>
                  <circle cx="15" cy="9.5" r="1.2" fill="url(#hero-g)"/>
                  <path d="M2 12h3M19 12h3M12 2v3M12 19v3" stroke="rgba(167,139,250,0.3)" stroke-width="0.8" stroke-linecap="round"/>
                  <defs><linearGradient id="hero-g" x1="0" y1="0" x2="24" y2="24"><stop stop-color="#a78bfa"/><stop offset="1" stop-color="#4facfe"/></linearGradient></defs>
                </svg>
              </div>
            </div>
            <h2 class="diff-title">选择版本</h2>
            <p class="diff-desc">温和版适合朋友聚会，火辣版适合亲密好友</p>
          </div>
          <div class="diff-cards">
            <button class="diff-card mild-card stagger-1" @click="selectDifficulty('mild')">
              <div class="dc-accent mild-accent"></div>
              <div class="dc-body">
                <div class="dc-icon-box mild-icon-box">
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                    <path d="M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20z" stroke="#4facfe" stroke-width="1.5" opacity="0.5"/>
                    <path d="M8 14s1.5 2 4 2 4-2 4-2" stroke="#4facfe" stroke-width="1.5" stroke-linecap="round"/>
                    <circle cx="9" cy="10" r="1" fill="#4facfe"/>
                    <circle cx="15" cy="10" r="1" fill="#4facfe"/>
                  </svg>
                </div>
                <div class="dc-info">
                  <div class="dc-name">温和版</div>
                  <div class="dc-sub">朋友聚会 · 轻松愉快</div>
                </div>
                <svg class="dc-arrow" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 18l6-6-6-6"/></svg>
              </div>
            </button>
            <button class="diff-card spicy-card stagger-2" @click="selectDifficulty('spicy')">
              <div class="dc-accent spicy-accent"></div>
              <div class="dc-body">
                <div class="dc-icon-box spicy-icon-box">
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                    <path d="M12 23c-3.5 0-7-2.5-7-8 0-4 3-7.5 4.5-9 .3-.3.8-.1.8.3-.1 1.5.5 3.2 1.7 4.2.2.2.5.1.6-.1.5-1.2.8-2.8.5-4.5-.1-.4.4-.6.7-.4C16.5 7.5 19 11 19 15c0 5.5-3.5 8-7 8z" stroke="#ff6b6b" stroke-width="1.5" fill="none"/>
                    <path d="M12 23c-1.5 0-3.5-1.5-3.5-4.5 0-2 1.5-3.5 2.5-4.5.2-.2.5 0 .5.2 0 .8.3 1.5.8 2 .1.1.3.1.3 0 .3-.6.4-1.3.3-2 0-.2.2-.3.4-.2 1.3 1 2.2 2.5 2.2 4.5 0 3-2 4.5-3.5 4.5z" fill="url(#fire-g)" opacity="0.3"/>
                    <defs><linearGradient id="fire-g" x1="12" y1="14" x2="12" y2="23"><stop stop-color="#ffd700"/><stop offset="1" stop-color="#ff6b6b"/></linearGradient></defs>
                  </svg>
                </div>
                <div class="dc-info">
                  <div class="dc-name">火辣版</div>
                  <div class="dc-sub">亲密好友 · 劲爆刺激</div>
                </div>
                <svg class="dc-arrow" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 18l6-6-6-6"/></svg>
              </div>
            </button>
          </div>
        </div>

        <!-- 第二步：真心话/大冒险选择 -->
        <div v-else-if="!mode" key="select" class="mode-select">
          <div class="mode-badge-wrap">
            <span class="mode-badge" :class="difficulty">
              {{ difficulty === 'mild' ? '温和版' : '火辣版' }}
            </span>
          </div>
          <div class="mode-hero-text">选择挑战方式</div>
          <div class="mode-duo">
            <button class="duo-card truth-duo stagger-1" @click="selectMode('truth')">
              <div class="duo-glow truth-glow-bg"></div>
              <div class="duo-icon-wrap">
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="12" r="10" stroke="#4facfe" stroke-width="1" opacity="0.25"/>
                  <circle cx="12" cy="12" r="7" stroke="#4facfe" stroke-width="0.8" opacity="0.15"/>
                  <path d="M12 7c-2.2 0-3.5 1.2-3.5 3 0 .5.4.9.9.9s.9-.4.9-.9c0-.8.6-1.3 1.7-1.3s1.7.6 1.7 1.5c0 .6-.3 1-.9 1.5l-.6.4c-.7.6-1 1.2-1 2.1v.3c0 .5.4.9.9.9s.9-.4.9-.9v-.1c0-.5.2-.8.7-1.2l.6-.5c.9-.7 1.4-1.5 1.4-2.6C15.6 8.3 14.2 7 12 7z" fill="url(#truth-ig)"/>
                  <circle cx="12" cy="17.5" r="1" fill="url(#truth-ig)"/>
                  <defs><linearGradient id="truth-ig" x1="8" y1="7" x2="16" y2="18"><stop stop-color="#4facfe"/><stop offset="1" stop-color="#00f2fe"/></linearGradient></defs>
                </svg>
              </div>
              <div class="duo-name">真心话</div>
              <div class="duo-sub">灵魂拷问</div>
            </button>
            <div class="duo-vs">
              <span class="vs-text">VS</span>
            </div>
            <button class="duo-card dare-duo stagger-2" @click="selectMode('dare')">
              <div class="duo-glow dare-glow-bg"></div>
              <div class="duo-icon-wrap">
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="12" r="10" stroke="#fa709a" stroke-width="1" opacity="0.25"/>
                  <circle cx="12" cy="12" r="7" stroke="#fa709a" stroke-width="0.8" opacity="0.15"/>
                  <path d="M13 3l-1.5 9h4L10 21l1.5-9H7.5L13 3z" fill="url(#dare-ig)" opacity="0.9"/>
                  <defs><linearGradient id="dare-ig" x1="7" y1="3" x2="16" y2="21"><stop stop-color="#fa709a"/><stop offset="1" stop-color="#fee140"/></linearGradient></defs>
                </svg>
              </div>
              <div class="duo-name">大冒险</div>
              <div class="duo-sub">勇气挑战</div>
            </button>
          </div>
        </div>

        <!-- 第三步：游戏结果 -->
        <div v-else key="result" class="game-result">
          <div class="result-tags">
            <span class="result-badge" :class="difficulty">
              {{ difficulty === 'mild' ? '温和' : '火辣' }}
            </span>
            <span class="result-badge" :class="mode">
              {{ mode === 'truth' ? '真心话' : '大冒险' }}
            </span>
          </div>

          <div class="card-area" :class="{ animating: isAnimating }">
            <Transition name="card-flip" mode="out-in">
              <div v-if="!isRevealed" key="front" class="td-card td-card-front" :class="mode" @click="reveal">
                <div class="card-pattern"></div>
                <div class="card-shimmer"></div>
                <div class="cover-icon-wrap">
                  <svg width="52" height="52" viewBox="0 0 24 24" fill="none">
                    <circle cx="12" cy="12" r="10.5" :stroke="mode === 'truth' ? '#4facfe' : '#fa709a'" stroke-width="0.8" stroke-dasharray="3 3" opacity="0.3">
                      <animateTransform attributeName="transform" type="rotate" from="0 12 12" to="360 12 12" dur="20s" repeatCount="indefinite"/>
                    </circle>
                    <circle cx="12" cy="12" r="7" :stroke="mode === 'truth' ? '#4facfe' : '#fa709a'" stroke-width="0.6" opacity="0.15"/>
                    <path d="M12 8v4l2.5 2.5" :stroke="mode === 'truth' ? '#4facfe' : '#fa709a'" stroke-width="1.5" stroke-linecap="round" opacity="0.6"/>
                    <circle cx="12" cy="12" r="1.5" :fill="mode === 'truth' ? '#4facfe' : '#fa709a'" opacity="0.5"/>
                  </svg>
                </div>
                <div class="cover-text">点击揭晓</div>
                <div class="cover-hint">轻触卡片查看内容</div>
              </div>
              <div v-else key="back" class="td-card td-card-back" :class="mode">
                <div class="card-accent" :class="mode + '-accent'"></div>
                <div class="card-content">{{ currentContent }}</div>
              </div>
            </Transition>
          </div>

          <div class="result-actions">
            <button class="change-btn" @click="backToModeSelect">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M19 12H5M12 19l-7-7 7-7"/></svg>
              换一种
            </button>
            <button class="next-btn" :class="mode" @click="nextRound" :disabled="!isRevealed">
              <span class="btn-shimmer"></span>
              下一题
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 12h14M12 5l7 7-7 7"/></svg>
            </button>
          </div>
        </div>
      </Transition>
    </main>
  </div>
</template>

<style scoped>
/* ===== 容器 & 背景 ===== */
.truth-dare-mobile {
  position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: linear-gradient(160deg, #0f0c29 0%, #1a1040 45%, #120e2a 100%);
  display: flex; flex-direction: column; overflow: hidden;
  padding-bottom: env(safe-area-inset-bottom);
}
.td-bg { position: fixed; top: 0; left: 0; right: 0; bottom: 0; pointer-events: none; z-index: 0; }
.td-mesh { position: absolute; top: 0; left: 0; right: 0; bottom: 0; }
.td-mesh-1 {
  background: radial-gradient(ellipse 55% 45% at 20% 15%, rgba(79,172,254,0.18), transparent),
              radial-gradient(ellipse 50% 40% at 80% 30%, rgba(167,139,250,0.15), transparent);
  animation: tdDrift1 22s ease-in-out infinite alternate;
}
.td-mesh-2 {
  background: radial-gradient(ellipse 45% 50% at 65% 70%, rgba(250,112,154,0.12), transparent),
              radial-gradient(ellipse 40% 30% at 30% 85%, rgba(79,172,254,0.08), transparent);
  animation: tdDrift2 26s ease-in-out infinite alternate;
}
@keyframes tdDrift1 { 0% { transform: translate(0,0) scale(1); } 100% { transform: translate(15px,-10px) scale(1.04); } }
@keyframes tdDrift2 { 0% { transform: translate(0,0) scale(1); } 100% { transform: translate(-12px,15px) scale(1.06); } }
.td-grain {
  position: fixed; top: -50%; left: -50%; width: 200%; height: 200%;
  pointer-events: none; z-index: 1; opacity: 0.022;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  background-size: 128px 128px;
}

/* ===== Header ===== */
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
  background: linear-gradient(135deg, #4facfe 0%, #a78bfa 50%, #fa709a 100%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.header-spacer { width: 44px; }

.game-area {
  flex: 1; overflow-y: auto; padding: 20px 16px;
  padding-bottom: max(24px, env(safe-area-inset-bottom));
  display: flex; flex-direction: column; position: relative; z-index: 2;
}

/* ===== 页面过渡 ===== */
.page-fade-enter-active { animation: pageFadeIn 0.4s cubic-bezier(0.22,1,0.36,1); }
.page-fade-leave-active { animation: pageFadeOut 0.2s ease-out; }
@keyframes pageFadeIn { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: translateY(0); } }
@keyframes pageFadeOut { from { opacity: 1; } to { opacity: 0; transform: translateY(-10px); } }

.stagger-1 { animation: staggerIn 0.5s cubic-bezier(0.22,1,0.36,1) 0.06s both; }
.stagger-2 { animation: staggerIn 0.5s cubic-bezier(0.22,1,0.36,1) 0.16s both; }
@keyframes staggerIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }

/* ========== 第一步：难度选择 ========== */
.diff-select {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  max-width: 360px; margin: 0 auto; width: 100%;
}
.diff-hero { text-align: center; margin-bottom: 36px; }
.diff-icon-wrap {
  position: relative; width: 80px; height: 80px; margin: 0 auto 14px;
  display: flex; align-items: center; justify-content: center;
}
.diff-icon-inner {
  position: relative; z-index: 2;
  width: 64px; height: 64px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(167,139,250,0.06);
  border: 1px solid rgba(167,139,250,0.12);
  border-radius: 20px;
  animation: iconFloat 3.5s ease-in-out infinite;
}
@keyframes iconFloat { 0%,100% { transform: translateY(0); } 50% { transform: translateY(-6px); } }
.diff-icon-glow {
  position: absolute; top: 50%; left: 50%; width: 70px; height: 70px; transform: translate(-50%,-50%);
  background: radial-gradient(circle, rgba(167,139,250,0.3), rgba(79,172,254,0.15), transparent 70%);
  border-radius: 50%; filter: blur(14px); z-index: 0;
}
.diff-title { font-size: 1.4rem; font-weight: 800; color: rgba(255,255,255,0.92); margin: 0 0 6px; }
.diff-desc { font-size: 0.82rem; color: rgba(255,255,255,0.35); margin: 0; line-height: 1.5; }

.diff-cards { display: flex; flex-direction: column; gap: 12px; width: 100%; }

/* 难度卡片 — 横向条形，左侧色带 */
.diff-card {
  display: flex; align-items: stretch; border: none; cursor: pointer; border-radius: 18px;
  background: rgba(255,255,255,0.03); overflow: hidden; transition: all 0.15s ease;
  touch-action: manipulation; text-align: left; width: 100%;
  border: 1px solid rgba(255,255,255,0.06);
}
.diff-card:active { transform: scale(0.97); background: rgba(255,255,255,0.06); }

.dc-accent { width: 5px; flex-shrink: 0; }
.dc-accent.mild-accent { background: linear-gradient(180deg, #4facfe, #43e97b); }
.dc-accent.spicy-accent { background: linear-gradient(180deg, #ff6b6b, #ffd700); }

.dc-body {
  flex: 1; display: flex; align-items: center; gap: 14px; padding: 18px 16px;
}
.dc-icon-box {
  width: 44px; height: 44px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  border-radius: 12px;
}
.dc-icon-box.mild-icon-box {
  background: rgba(79,172,254,0.08);
  border: 1px solid rgba(79,172,254,0.15);
}
.dc-icon-box.spicy-icon-box {
  background: rgba(255,107,107,0.08);
  border: 1px solid rgba(255,107,107,0.15);
}
.dc-info { flex: 1; }
.dc-name { font-size: 1.1rem; font-weight: 700; color: rgba(255,255,255,0.9); margin-bottom: 2px; }
.dc-sub { font-size: 0.78rem; color: rgba(255,255,255,0.35); }
.dc-arrow { color: rgba(255,255,255,0.2); flex-shrink: 0; }

/* ========== 第二步：真心话/大冒险选择 ========== */
.mode-select {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  max-width: 380px; margin: 0 auto; width: 100%;
}
.mode-badge-wrap { margin-bottom: 12px; }
.mode-badge {
  display: inline-block; padding: 6px 16px; border-radius: 14px;
  font-size: 0.82rem; font-weight: 600;
}
.mode-badge.mild { background: rgba(79,172,254,0.1); color: #4facfe; border: 1px solid rgba(79,172,254,0.15); }
.mode-badge.spicy { background: rgba(255,107,107,0.1); color: #ff6b6b; border: 1px solid rgba(255,107,107,0.15); }

.mode-hero-text {
  font-size: 1rem; color: rgba(255,255,255,0.4); margin-bottom: 36px; letter-spacing: 1px;
}

/* 双卡 VS 布局 */
.mode-duo {
  display: flex; align-items: center; gap: 12px; width: 100%;
}

.duo-card {
  flex: 1; border: none; cursor: pointer; border-radius: 22px; padding: 28px 14px; text-align: center;
  position: relative; overflow: hidden; touch-action: manipulation; transition: all 0.15s ease;
}
.duo-card:active { transform: scale(0.96); }

.duo-card.truth-duo {
  background: rgba(79,172,254,0.06); border: 1.5px solid rgba(79,172,254,0.18);
}
.duo-card.truth-duo:active { box-shadow: 0 0 32px rgba(79,172,254,0.15); }

.duo-card.dare-duo {
  background: rgba(250,112,154,0.06); border: 1.5px solid rgba(250,112,154,0.18);
}
.duo-card.dare-duo:active { box-shadow: 0 0 32px rgba(250,112,154,0.15); }

.duo-glow {
  position: absolute; width: 100px; height: 100px; border-radius: 50%;
  filter: blur(40px); opacity: 0.25; pointer-events: none; top: -20px; right: -20px;
}
.truth-glow-bg { background: #4facfe; }
.dare-glow-bg { background: #fa709a; }

.duo-icon-wrap {
  width: 64px; height: 64px; margin: 0 auto 12px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: 18px;
  position: relative; z-index: 1;
}
.truth-duo .duo-icon-wrap {
  background: rgba(79,172,254,0.06);
  border-color: rgba(79,172,254,0.12);
}
.dare-duo .duo-icon-wrap {
  background: rgba(250,112,154,0.06);
  border-color: rgba(250,112,154,0.12);
}
.duo-name { font-size: 1.2rem; font-weight: 700; color: rgba(255,255,255,0.9); position: relative; z-index: 1; }
.duo-sub { font-size: 0.75rem; color: rgba(255,255,255,0.35); margin-top: 4px; position: relative; z-index: 1; }

.duo-vs {
  width: 40px; height: 40px; border-radius: 50%; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.08);
}
.vs-text {
  font-size: 0.7rem; font-weight: 800; color: rgba(255,255,255,0.25); letter-spacing: 1px;
}

/* ========== 第三步：结果 ========== */
.game-result {
  flex: 1; display: flex; flex-direction: column; justify-content: center; align-items: center; padding: 10px 0;
}
.result-tags { display: flex; gap: 8px; margin-bottom: 22px; }
.result-badge {
  padding: 6px 16px; border-radius: 14px; font-size: 0.82rem; font-weight: 600;
}
.result-badge.mild { background: rgba(79,172,254,0.1); color: #4facfe; border: 1px solid rgba(79,172,254,0.15); }
.result-badge.spicy { background: rgba(255,107,107,0.1); color: #ff6b6b; border: 1px solid rgba(255,107,107,0.15); }
.result-badge.truth { background: rgba(79,172,254,0.1); color: #4facfe; border: 1px solid rgba(79,172,254,0.15); }
.result-badge.dare { background: rgba(250,112,154,0.1); color: #fa709a; border: 1px solid rgba(250,112,154,0.15); }

/* ===== 翻牌 ===== */
.card-area { width: 100%; max-width: 340px; margin-bottom: 28px; }
.card-area.animating { animation: cardPulse 0.4s ease; }
@keyframes cardPulse { 0%,100% { transform: scale(1); } 50% { transform: scale(0.96); } }

.card-flip-enter-active { animation: cardFlipIn 0.4s cubic-bezier(0.22,1,0.36,1); }
.card-flip-leave-active { animation: cardFlipOut 0.25s ease-out; }
@keyframes cardFlipIn { 0% { opacity: 0; transform: scale(0.88) rotateY(8deg); } 100% { opacity: 1; transform: scale(1) rotateY(0); } }
@keyframes cardFlipOut { 0% { opacity: 1; transform: scale(1); } 100% { opacity: 0; transform: scale(0.88) rotateY(-8deg); } }

.td-card {
  width: 100%; min-height: 240px; border-radius: 20px;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  overflow: hidden; position: relative;
}

.td-card-front {
  cursor: pointer; touch-action: manipulation;
  background: rgba(15,12,41,0.75); backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px);
  border: 1.5px solid rgba(255,255,255,0.08); box-shadow: 0 8px 32px rgba(0,0,0,0.3);
}
.td-card-front.truth { border-color: rgba(79,172,254,0.2); box-shadow: 0 8px 32px rgba(0,0,0,0.3), 0 0 24px rgba(79,172,254,0.06); }
.td-card-front.dare { border-color: rgba(250,112,154,0.2); box-shadow: 0 8px 32px rgba(0,0,0,0.3), 0 0 24px rgba(250,112,154,0.06); }
.td-card-front:active { transform: scale(0.98); }

.card-pattern {
  position: absolute; inset: 0; opacity: 0.04; pointer-events: none;
  background-image: radial-gradient(circle at 25% 25%, #fff 1px, transparent 1px), radial-gradient(circle at 75% 75%, #fff 1px, transparent 1px);
  background-size: 24px 24px;
}
.card-shimmer {
  position: absolute; top: 0; left: -100%; width: 60%; height: 100%; pointer-events: none;
  background: linear-gradient(105deg, transparent 35%, rgba(255,255,255,0.06) 50%, transparent 65%);
  animation: cardShimmerSweep 4s cubic-bezier(0.22,1,0.36,1) infinite;
}
@keyframes cardShimmerSweep { 0% { left: -100%; } 30% { left: 160%; } 100% { left: 160%; } }

.cover-icon-wrap {
  position: relative; z-index: 1; animation: bounceUp 2s cubic-bezier(0.36,0,0.66,1) infinite;
}
@keyframes bounceUp { 0%,100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
.cover-text { font-size: 1.1rem; color: rgba(255,255,255,0.45); margin-top: 12px; font-weight: 600; letter-spacing: 1px; position: relative; z-index: 1; }
.cover-hint { font-size: 0.72rem; color: rgba(255,255,255,0.2); margin-top: 6px; position: relative; z-index: 1; }

.td-card-back {
  background: rgba(15,12,41,0.8); backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px);
  padding: 36px 28px;
}
.td-card-back.truth { border: 1.5px solid rgba(79,172,254,0.25); box-shadow: 0 12px 48px rgba(0,0,0,0.35), 0 0 40px rgba(79,172,254,0.08); }
.td-card-back.dare { border: 1.5px solid rgba(250,112,154,0.25); box-shadow: 0 12px 48px rgba(0,0,0,0.35), 0 0 40px rgba(250,112,154,0.08); }

.card-accent { position: absolute; top: 0; left: 0; width: 4px; height: 100%; border-radius: 4px 0 0 4px; }
.truth-accent { background: linear-gradient(180deg, #4facfe, #00f2fe); }
.dare-accent { background: linear-gradient(180deg, #fa709a, #fee140); }

.card-content {
  font-size: 1.4rem; line-height: 1.85; letter-spacing: 0.3px;
  color: rgba(255,255,255,0.92); text-align: center;
  animation: contentReveal 0.4s cubic-bezier(0.22,1,0.36,1) 0.1s both;
}
@keyframes contentReveal { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }

/* ===== 按钮 ===== */
.result-actions { display: flex; gap: 14px; width: 100%; max-width: 340px; }
.change-btn {
  flex: 1; height: 52px; display: flex; align-items: center; justify-content: center; gap: 8px;
  background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.08);
  border-radius: 18px; color: rgba(255,255,255,0.6); font-size: 0.95rem; font-weight: 500;
  cursor: pointer; transition: all 0.15s ease;
}
.change-btn:active { background: rgba(255,255,255,0.1); transform: scale(0.97); }

.next-btn {
  flex: 1.5; height: 52px; display: flex; align-items: center; justify-content: center; gap: 8px;
  border: none; border-radius: 18px; font-size: 1rem; font-weight: 600;
  cursor: pointer; position: relative; overflow: hidden; transition: all 0.15s ease;
}
.next-btn.truth { background: linear-gradient(135deg, #4facfe, #00f2fe); color: #0a0a14; box-shadow: 0 4px 20px rgba(79,172,254,0.25); }
.next-btn.dare { background: linear-gradient(135deg, #fa709a, #fee140); color: #0a0a14; box-shadow: 0 4px 20px rgba(250,112,154,0.25); }
.next-btn:active:not(:disabled) { transform: scale(0.97); }
.next-btn:disabled { opacity: 0.35; cursor: not-allowed; box-shadow: none; }
.next-btn:disabled .btn-shimmer { display: none; }
.btn-shimmer {
  position: absolute; top: 0; left: -100%; width: 60%; height: 100%; pointer-events: none;
  background: linear-gradient(105deg, transparent 35%, rgba(255,255,255,0.35) 50%, transparent 65%);
  animation: shimmerSweep 2.8s cubic-bezier(0.22,1,0.36,1) infinite;
}
@keyframes shimmerSweep { 0% { left: -100%; } 40% { left: 160%; } 100% { left: 160%; } }

/* ===== 响应式 ===== */
@media (max-height: 700px) {
  .diff-hero { margin-bottom: 24px; }
  .diff-icon-inner { width: 52px; height: 52px; }
  .diff-icon-inner svg { width: 36px; height: 36px; }
  .duo-card { padding: 20px 12px; }
  .duo-icon-wrap { width: 52px; height: 52px; }
  .duo-icon-wrap svg { width: 40px; height: 40px; }
  .td-card { min-height: 180px; }
  .card-content { font-size: 1.2rem; line-height: 1.7; }
}
@media (min-width: 500px) {
  .diff-cards { max-width: 380px; }
  .mode-duo { max-width: 400px; }
  .card-area { max-width: 400px; }
  .result-actions { max-width: 400px; }
}
</style>
