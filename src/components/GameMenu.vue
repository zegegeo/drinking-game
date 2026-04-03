<script setup>
import { ref } from 'vue'

defineProps({
  games: {
    type: Object,
    required: true
  }
})

defineEmits(['selectGame'])

const triggerHaptic = () => {
  if ('vibrate' in navigator) navigator.vibrate(20)
}

const themes = [
  { g1: '#a78bfa', g2: '#6366f1' },  // 真心话大冒险 紫→蓝
  { g1: '#fb923c', g2: '#ef4444' },  // 掷骰子 橙→红
  { g1: '#f43f5e', g2: '#ec4899' },  // 数字炸弹 红→粉
  { g1: '#34d399', g2: '#06b6d4' },  // 幸运大转盘 绿→青
  { g1: '#818cf8', g2: '#a78bfa' },  // 谁是卧底 靛→紫
  { g1: '#10b981', g2: '#14b8a6' },  // 21点 翠绿→青
  { g1: '#f472b6', g2: '#c084fc' },  // 小姐牌 粉→紫
  { g1: '#fbbf24', g2: '#f59e0b' },  // 国王游戏 金→橙
  { g1: '#38bdf8', g2: '#22d3ee' },  // 手速挑战 蓝→青
  { g1: '#e879f9', g2: '#f472b6' },  // 猜大小 紫红→粉
]

const getTheme = (index) => themes[index % themes.length]
</script>

<template>
  <div class="bento-home">
    <!-- 背景 -->
    <div class="bg-layer">
      <div class="bg-orb bg-orb-1"></div>
      <div class="bg-orb bg-orb-2"></div>
      <div class="bg-orb bg-orb-3"></div>
    </div>

    <!-- Header -->
    <header class="menu-header">
      <div class="header-inner">
        <span class="header-emoji">🍻</span>
        <h1 class="header-title">喝酒小游戏</h1>
        <p class="header-sub">选一个开始玩吧</p>
      </div>
    </header>

    <!-- Bento Grid -->
    <main class="bento-scroll">
      <div class="bento-grid">
        <button
          v-for="(game, key, index) in games"
          :key="key"
          class="bento-card"
          :class="{ 'card-tall': index < 2 }"
          :style="{
            '--g1': getTheme(index).g1,
            '--g2': getTheme(index).g2,
            '--delay': (index * 0.06) + 's'
          }"
          @click="$emit('selectGame', key); triggerHaptic()"
        >
          <div class="card-bg"></div>
          <div class="card-glow"></div>
          <div class="card-content">
            <span class="card-icon">{{ game.icon }}</span>
            <div class="card-name">{{ game.name }}</div>
            <div class="card-desc">{{ game.description }}</div>
          </div>
        </button>
      </div>
    </main>
  </div>
</template>

<style scoped>
/* ===== 容器 ===== */
.bento-home {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: linear-gradient(160deg, #0f0c29 0%, #1a1040 45%, #120e2a 100%);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
}

/* ===== 背景光斑 ===== */
.bg-layer {
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
}

.bg-orb-1 {
  width: 340px; height: 340px;
  top: -100px; left: -80px;
  background: rgba(167, 139, 250, 0.12);
  animation: orbFloat1 18s ease-in-out infinite alternate;
}

.bg-orb-2 {
  width: 280px; height: 280px;
  bottom: -80px; right: -60px;
  background: rgba(244, 114, 182, 0.1);
  animation: orbFloat2 22s ease-in-out infinite alternate;
}

.bg-orb-3 {
  width: 200px; height: 200px;
  top: 40%; left: 50%;
  transform: translateX(-50%);
  background: rgba(56, 189, 248, 0.06);
  animation: orbFloat3 20s ease-in-out infinite alternate;
}

@keyframes orbFloat1 {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(40px, 30px) scale(1.1); }
}
@keyframes orbFloat2 {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(-30px, -40px) scale(1.08); }
}
@keyframes orbFloat3 {
  0% { transform: translateX(-50%) translateY(0) scale(1); }
  100% { transform: translateX(-50%) translateY(-30px) scale(1.15); }
}

/* ===== Header ===== */
.menu-header {
  position: relative;
  z-index: 3;
  padding: 20px 20px 10px;
  padding-top: max(20px, calc(env(safe-area-inset-top) + 10px));
  text-align: center;
}

.header-inner {
  animation: headerIn 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes headerIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

.header-emoji {
  font-size: 2.2rem;
  display: block;
  margin-bottom: 6px;
}

.header-title {
  font-size: 1.5rem;
  font-weight: 800;
  margin: 0 0 4px;
  letter-spacing: 1px;
  background: linear-gradient(135deg, #e0e7ff 0%, #c4b5fd 40%, #f9a8d4 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header-sub {
  font-size: 0.82rem;
  color: rgba(255, 255, 255, 0.35);
  margin: 0;
  font-weight: 400;
}

/* ===== Bento Grid 滚动区 ===== */
.bento-scroll {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 12px 14px 20px;
  padding-bottom: max(20px, calc(env(safe-area-inset-bottom) + 12px));
  -webkit-overflow-scrolling: touch;
  position: relative;
  z-index: 2;
}

.bento-scroll::-webkit-scrollbar { width: 0; display: none; }

.bento-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  max-width: 420px;
  margin: 0 auto;
}

/* ===== 卡片 ===== */
.bento-card {
  position: relative;
  border: none;
  cursor: pointer;
  border-radius: 20px;
  padding: 22px 16px 18px;
  text-align: center;
  overflow: hidden;
  touch-action: manipulation;
  transition: transform 0.2s cubic-bezier(0.22, 1, 0.36, 1), box-shadow 0.2s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 140px;

  /* 入场动画 */
  opacity: 0;
  animation: cardEnter 0.5s cubic-bezier(0.22, 1, 0.36, 1) forwards;
  animation-delay: var(--delay, 0s);
}

.bento-card.card-tall {
  min-height: 170px;
}

@keyframes cardEnter {
  from { opacity: 0; transform: translateY(20px) scale(0.95); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}

/* 卡片暗底 + 渐变叠加 */
.card-bg {
  position: absolute;
  inset: 0;
  border-radius: 20px;
  background: rgba(15, 12, 35, 0.85);
}

/* 渐变色叠在暗底上 */
.bento-card::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 20px;
  background: linear-gradient(145deg, var(--g1), var(--g2));
  opacity: 0.14;
  pointer-events: none;
  transition: opacity 0.2s ease;
}

.bento-card:active::after {
  opacity: 0.26;
}

/* 卡片边框：磨砂玻璃 */
.bento-card::before {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  pointer-events: none;
  transition: border-color 0.2s ease;
}

.bento-card:active::before {
  border-color: rgba(255, 255, 255, 0.18);
}

/* 卡片内部发光 */
.card-glow {
  position: absolute;
  top: -30px;
  right: -30px;
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--g1), var(--g2));
  opacity: 0.1;
  filter: blur(30px);
  pointer-events: none;
  transition: opacity 0.2s ease;
}

.bento-card:active .card-glow {
  opacity: 0.2;
}

/* 按下效果 */
.bento-card:active {
  transform: scale(0.96);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

/* 卡片内容 */
.card-content {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.card-icon {
  font-size: 2.4rem;
  margin-bottom: 8px;
  display: block;
  line-height: 1;
}

.card-tall .card-icon {
  font-size: 2.8rem;
  margin-bottom: 10px;
}

.card-name {
  font-size: 1.05rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.92);
  letter-spacing: 0.5px;
  margin-bottom: 4px;
}

.card-desc {
  font-size: 0.72rem;
  color: rgba(255, 255, 255, 0.4);
  line-height: 1.45;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* ===== 小屏适配 ===== */
@media (max-height: 640px) {
  .menu-header { padding: 12px 20px 6px; }
  .header-emoji { font-size: 1.6rem; margin-bottom: 4px; }
  .header-title { font-size: 1.2rem; }
  .bento-card { min-height: 120px; padding: 16px 12px 14px; border-radius: 16px; }
  .bento-card.card-tall { min-height: 140px; }
  .card-icon { font-size: 2rem; margin-bottom: 6px; }
  .card-tall .card-icon { font-size: 2.2rem; }
  .card-name { font-size: 0.92rem; }
  .bento-grid { gap: 8px; }
}

/* ===== 大屏适配 ===== */
@media (min-width: 500px) {
  .bento-grid { max-width: 440px; }
}
</style>
