<script setup lang="ts">
import { computed } from 'vue'
import type { GameSummary } from '@/types/game'

interface Props {
  show: boolean
  finalTurn: number
  highScore: number
  isNewRecord: boolean
  lastGame: GameSummary | null
  finalHealth: number
  finalHunger: number
  finalThirst: number
  finalWood: number
  finalStone: number
}

const props = defineProps<Props>()

const emit = defineEmits<{
  restart: []
}>()

interface StatCompare {
  label: string
  current: number
  last?: number
  icon: string
  color: string
}

const statComparisons = computed<StatCompare[]>(() => {
  const stats: StatCompare[] = [
    { label: '生存回合', current: props.finalTurn, last: props.lastGame?.turn, icon: '⏱️', color: 'text-white' },
    { label: '生命值', current: props.finalHealth, last: props.lastGame?.finalHealth, icon: '❤️', color: 'text-red-400' },
    { label: '饥饿值', current: props.finalHunger, last: props.lastGame?.finalHunger, icon: '🍖', color: 'text-orange-400' },
    { label: '口渴值', current: props.finalThirst, last: props.lastGame?.finalThirst, icon: '💧', color: 'text-blue-400' },
    { label: '木材', current: props.finalWood, last: props.lastGame?.finalWood, icon: '🪵', color: 'text-amber-600' },
    { label: '石头', current: props.finalStone, last: props.lastGame?.finalStone, icon: '🪨', color: 'text-gray-400' },
  ]
  return stats
})

function getDiff(current: number, last?: number): number | null {
  if (last === undefined) return null
  return current - last
}

function getDiffClass(diff: number | null, label: string): string {
  if (diff === null || diff === 0) return 'text-gray-500'
  if (label === '饥饿值' || label === '口渴值') {
    return diff < 0 ? 'text-green-400' : 'text-red-400'
  }
  return diff > 0 ? 'text-green-400' : 'text-red-400'
}

function getDiffSign(diff: number | null): string {
  if (diff === null || diff === 0) return '—'
  return diff > 0 ? `+${diff}` : `${diff}`
}
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div
        v-if="show"
        class="fixed inset-0 z-50 flex items-center justify-center p-4"
      >
        <div class="absolute inset-0 bg-black/70 backdrop-blur-sm"></div>
        <div class="relative bg-game-card rounded-3xl p-8 max-w-lg w-full border border-game-border shadow-2xl animate-slide-up max-h-[90vh] overflow-y-auto">
          <div class="text-center">
            <div class="text-6xl mb-4">💀</div>
            <h2 class="text-3xl font-bold text-white mb-2">游戏结束</h2>
            <p class="text-gray-400 mb-6">你没能在荒野中生存下来...</p>

            <div class="bg-gray-800/50 rounded-2xl p-5 mb-4 space-y-3">
              <div class="flex justify-between items-center">
                <span class="text-gray-400">生存回合</span>
                <span class="text-2xl font-bold text-white">{{ finalTurn }}</span>
              </div>
              <div class="border-t border-gray-700"></div>
              <div class="flex justify-between items-center">
                <span class="text-gray-400">最高纪录</span>
                <span class="text-xl font-bold" :class="isNewRecord ? 'text-yellow-400' : 'text-gray-300'">
                  {{ highScore }}
                  <span v-if="isNewRecord" class="text-sm ml-1">🏆 新纪录！</span>
                </span>
              </div>
            </div>

            <div v-if="lastGame" class="bg-gray-800/50 rounded-2xl p-5 mb-6">
              <h3 class="text-sm font-semibold text-purple-400 mb-4 flex items-center gap-2">
                <span>📊</span>
                <span>与上一局对比</span>
              </h3>
              <div class="space-y-2.5">
                <div
                  v-for="stat in statComparisons"
                  :key="stat.label"
                  class="flex items-center justify-between text-sm"
                >
                  <div class="flex items-center gap-2">
                    <span>{{ stat.icon }}</span>
                    <span class="text-gray-400">{{ stat.label }}</span>
                  </div>
                  <div class="flex items-center gap-3">
                    <span :class="[stat.color, 'font-semibold tabular-nums']">{{ Math.round(stat.current) }}</span>
                    <span class="text-gray-600">vs</span>
                    <span class="text-gray-400 tabular-nums">{{ Math.round(stat.last ?? 0) }}</span>
                    <span
                      :class="[getDiffClass(getDiff(stat.current, stat.last), stat.label), 'font-medium tabular-nums w-12 text-right']"
                    >
                      {{ getDiffSign(getDiff(stat.current, stat.last)) }}
                    </span>
                  </div>
                </div>
              </div>
              <div class="mt-4 pt-4 border-t border-gray-700 text-xs text-gray-500">
                上一局：{{ lastGame.deathReason }}
              </div>
            </div>

            <button
              @click="emit('restart')"
              class="w-full py-4 px-6 bg-gradient-to-r from-green-600 to-emerald-600 hover:from-green-500 hover:to-emerald-500 text-white font-bold text-lg rounded-xl transition-all duration-200 hover:scale-[1.02] active:scale-[0.98] shadow-lg hover:shadow-green-500/25"
            >
              🔄 重新开始
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.modal-enter-active,
.modal-leave-active {
  transition: all 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .relative,
.modal-leave-to .relative {
  transform: scale(0.9);
}
</style>
