<script setup lang="ts">
import { computed } from 'vue'
import StatusPanel from '@/components/StatusPanel.vue'
import ActionButtons from '@/components/ActionButtons.vue'
import EventLog from '@/components/EventLog.vue'
import GameOverModal from '@/components/GameOverModal.vue'
import { useGame } from '@/composables/useGame'

const { state, highScore, lastGameSummary, canPerformAction, gatherWood, gatherStone, hunt, drink, restart } = useGame()

const isNewRecord = computed(() => state.value.turn >= highScore.value && state.value.turn > 0)

const turnCompare = computed(() => {
  if (!lastGameSummary.value) return null
  const diff = state.value.turn - lastGameSummary.value.turn
  if (diff === 0) return 0
  return diff
})

function formatTime(timestamp: number): string {
  const date = new Date(timestamp)
  return date.toLocaleString('zh-CN', {
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
  })
}
</script>

<template>
  <div class="min-h-screen bg-game-bg text-white">
    <div class="absolute inset-0 overflow-hidden pointer-events-none">
      <div class="absolute top-0 left-1/4 w-96 h-96 bg-green-900/10 rounded-full blur-3xl"></div>
      <div class="absolute bottom-0 right-1/4 w-96 h-96 bg-blue-900/10 rounded-full blur-3xl"></div>
    </div>

    <div class="relative z-10 max-w-6xl mx-auto px-4 py-8">
      <header class="text-center mb-8">
        <h1 class="text-4xl font-bold mb-2 bg-gradient-to-r from-green-400 via-emerald-400 to-teal-400 bg-clip-text text-transparent">
          🏕️ 荒野生存
        </h1>
        <p class="text-gray-400">在恶劣的荒野中尽可能生存更久</p>
      </header>

      <div class="flex justify-center gap-4 mb-8 flex-wrap">
        <div class="bg-game-card/80 backdrop-blur px-6 py-3 rounded-xl border border-game-border">
          <span class="text-gray-400 text-sm">当前回合</span>
          <p class="text-2xl font-bold text-white tabular-nums flex items-center gap-2">
            {{ state.turn }}
            <span
              v-if="turnCompare !== null && turnCompare !== 0"
              :class="turnCompare > 0 ? 'text-green-400' : 'text-red-400'"
              class="text-sm font-normal"
            >
              {{ turnCompare > 0 ? '↑' : '↓' }} {{ Math.abs(turnCompare) }}
            </span>
          </p>
        </div>
        <div class="bg-game-card/80 backdrop-blur px-6 py-3 rounded-xl border border-game-border">
          <span class="text-gray-400 text-sm">最高纪录</span>
          <p class="text-2xl font-bold text-yellow-400 tabular-nums">🏆 {{ highScore }}</p>
        </div>
        <div
          v-if="lastGameSummary"
          class="bg-game-card/80 backdrop-blur px-6 py-3 rounded-xl border border-game-border"
        >
          <span class="text-gray-400 text-sm">最近一局</span>
          <p class="text-2xl font-bold text-purple-400 tabular-nums">
            🕐 {{ lastGameSummary.turn }}
          </p>
          <p class="text-xs text-gray-500 mt-0.5">
            {{ lastGameSummary.deathReason }} · {{ formatTime(lastGameSummary.endTime) }}
          </p>
        </div>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div class="space-y-6">
          <StatusPanel
            :health="state.health"
            :hunger="state.hunger"
            :thirst="state.thirst"
            :wood="state.wood"
            :stone="state.stone"
          />
        </div>

        <div>
          <ActionButtons
            :can-gather-wood="canPerformAction('gatherWood')"
            :can-gather-stone="canPerformAction('gatherStone')"
            :can-hunt="canPerformAction('hunt')"
            :can-drink="canPerformAction('drink')"
            :disabled="state.isGameOver"
            @gather-wood="gatherWood"
            @gather-stone="gatherStone"
            @hunt="hunt"
            @drink="drink"
          />
        </div>

        <div>
          <EventLog :logs="state.logs" />
        </div>
      </div>

      <footer class="mt-8 text-center text-gray-500 text-sm">
        <p>💡 提示：生命值归零或饥饿/口渴值满格则游戏结束</p>
      </footer>
    </div>

    <GameOverModal
      :show="state.isGameOver"
      :final-turn="state.turn"
      :high-score="highScore"
      :is-new-record="isNewRecord"
      :last-game="lastGameSummary"
      :final-health="state.health"
      :final-hunger="state.hunger"
      :final-thirst="state.thirst"
      :final-wood="state.wood"
      :final-stone="state.stone"
      @restart="restart"
    />
  </div>
</template>
