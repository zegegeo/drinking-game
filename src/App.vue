<script setup>
import { ref, computed, shallowRef } from 'vue'
import GameMenu from './components/GameMenu.vue'
import TruthOrDare from './components/TruthOrDare.vue'
import DiceGame from './components/DiceGame.vue'
import NumberBomb from './components/NumberBomb.vue'
import LuckyWheel from './components/LuckyWheel.vue'
import UndercoverGame from './components/UndercoverGame.vue'
import BlackjackGame from './components/BlackjackGame.vue'
import CardDrawGame from './components/CardDrawGame.vue'
import KingGame from './components/KingGame.vue'
import ReactionGame from './components/ReactionGame.vue'
import HighLowGame from './components/HighLowGame.vue'

const currentGame = ref(null)

const gameComponents = {
  truthOrDare: TruthOrDare,
  diceGame: DiceGame,
  numberBomb: NumberBomb,
  luckyWheel: LuckyWheel,
  undercover: UndercoverGame,
  blackjack: BlackjackGame,
  cardDraw: CardDrawGame,
  kingGame: KingGame,
  reactionGame: ReactionGame,
  highLow: HighLowGame,
}

const games = {
  truthOrDare: {
    name: '真心话大冒险',
    icon: '🎭',
    description: '经典派对游戏，挑战你的勇气与坦诚',
    type: 'offline'
  },
  diceGame: {
    name: '掷骰子',
    icon: '🎲',
    description: '选择骰子数量，摇一摇看结果',
    type: 'offline'
  },
  numberBomb: {
    name: '数字炸弹',
    icon: '💣',
    description: '轮流猜数字，猜中炸弹的人喝酒！',
    type: 'offline'
  },
  luckyWheel: {
    name: '幸运大转盘',
    icon: '🎡',
    description: '转到什么就做什么，公平又刺激',
    type: 'offline'
  },
  undercover: {
    name: '谁是卧底',
    icon: '🕵️',
    description: '找出那个与众不同的玩家',
    type: 'offline'
  },
  blackjack: {
    name: '21点',
    icon: '🃏',
    description: '多人发牌比点数，爆牌的人喝酒！',
    type: 'offline'
  },
  cardDraw: {
    name: '小姐牌',
    icon: '🎴',
    description: '经典喝酒扑克，每张牌都有规则！',
    type: 'offline'
  },
  kingGame: {
    name: '国王游戏',
    icon: '👑',
    description: '随机选出国王，下达命令！',
    type: 'offline'
  },
  reactionGame: {
    name: '手速挑战',
    icon: '⚡',
    description: '比拼反应速度，最慢的人喝酒',
    type: 'offline'
  },
  highLow: {
    name: '猜大小',
    icon: '🔮',
    description: '翻牌猜大小，猜错就喝酒！',
    type: 'offline'
  }
}

const activeComponent = computed(() => {
  if (!currentGame.value) return null
  return gameComponents[currentGame.value] || null
})

const selectGame = (gameKey) => {
  currentGame.value = gameKey
}

const goBack = () => {
  currentGame.value = null
}
</script>

<template>
  <div class="app">
    <Transition name="fade" mode="out-in">
      <GameMenu
        v-if="!currentGame"
        :games="games"
        @select-game="selectGame"
      />
      <component
        v-else
        :is="activeComponent"
        :key="currentGame"
        @go-back="goBack"
      />
    </Transition>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
