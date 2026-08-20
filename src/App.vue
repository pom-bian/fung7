<script setup lang="ts">
import { computed, ref } from 'vue'

type ColorChoice = { hex: string; name: string }
const totalRounds = 10

const target = ref<ColorChoice>({ name: 'target', hex: '#ff7657' })
const choices = ref<ColorChoice[]>([])
const selected = ref<string | null>(null)
const score = ref(0)
const streak = ref(0)
const bestStreak = ref(0)
const round = ref(1)
const finished = ref(false)
const isCorrect = computed(() => selected.value === target.value.hex)
const resultText = computed(() => isCorrect.value ? 'Perfect match!' : 'Close one!')

function shuffle<T>(items: T[]) { return [...items].sort(() => Math.random() - 0.5) }
function toHex(value: number) { return Math.max(0, Math.min(255, value)).toString(16).padStart(2, '0') }
function makeHex(rgb: number[]) { return `#${rgb.map(toHex).join('')}` }
function newRound() {
  const base = [40, 80, 120].map((min) => min + Math.floor(Math.random() * 150))
  const correct = makeHex(base)
  const wrong = [5, 10, 16].map((distance) => ({
    hex: makeHex(base.map((channel) => channel + (Math.random() > .5 ? distance : -distance))),
    name: 'near match',
  }))
  target.value = { hex: correct, name: 'target' }
  choices.value = shuffle([{ hex: correct, name: 'exact match' }, ...wrong])
  selected.value = null
}
function choose(choice: ColorChoice) {
  if (selected.value) return
  selected.value = choice.hex
  if (choice.hex === target.value.hex) { streak.value += 1; bestStreak.value = Math.max(bestStreak.value, streak.value); score.value += 10 + Math.max(0, streak.value - 1) * 2 } else streak.value = 0
}
function nextRound() {
  if (round.value >= totalRounds) { finished.value = true; return }
  round.value += 1
  newRound()
}
function restartGame() {
  score.value = 0
  streak.value = 0
  bestStreak.value = 0
  round.value = 1
  finished.value = false
  newRound()
}
newRound()
</script>

<template>
  <main class="game-shell">
    <nav class="topbar" aria-label="Main navigation">
      <a class="brand" href="./" aria-label="Hue Hunt home"><span class="brand-mark"><i></i><i></i><i></i></span><span>hue hunt</span></a>
      <span class="nav-note">a tiny color game</span>
    </nav>
    <section class="game-card" aria-labelledby="game-title">
      <div v-if="!finished" class="game-heading">
        <div><p class="eyebrow">Round {{ String(round).padStart(2, '0') }}</p><h1 id="game-title">Find the <em>match.</em></h1><p class="subtitle">One of these colors is the same as the target. Trust your eyes.</p></div>
        <div class="scoreboard" aria-label="Current score"><div><span>score</span><strong>{{ score }}</strong></div><div><span>streak</span><strong>{{ streak }}<small>×</small></strong></div></div>
      </div>
      <div v-if="!finished" class="target-area"><div class="target-swatch" :style="{ backgroundColor: target.hex }" aria-label="Target color"></div><div class="target-copy"><span class="target-label">target color</span><strong>What do you see?</strong><span class="target-hint">Pick the closest swatch below</span></div></div>
      <div v-if="!finished" class="choices" role="group" aria-label="Color choices">
        <button v-for="(choice, index) in choices" :key="choice.hex" class="color-choice" :class="{ correct: selected && choice.hex === target.hex, wrong: selected === choice.hex && choice.hex !== target.hex }" :style="{ '--choice-color': choice.hex }" :aria-label="`Color choice ${index + 1}`" :disabled="!!selected" @click="choose(choice)"><span class="choice-number">0{{ index + 1 }}</span><span v-if="selected && choice.hex === target.hex" class="choice-status">✓</span><span v-else-if="selected === choice.hex" class="choice-status">×</span></button>
      </div>
      <div v-if="!finished" class="result" :class="{ visible: !!selected, miss: !!selected && !isCorrect }" aria-live="polite"><div><span class="result-kicker">{{ isCorrect ? 'That’s it' : 'The answer was' }}</span><strong>{{ resultText }}</strong></div><button v-if="selected" class="next-button" @click="nextRound">{{ round === totalRounds ? 'See score' : 'Next round' }} <span>→</span></button><span v-else class="waiting">Choose a color to begin</span></div>
      <div v-else class="final-result"><p class="eyebrow">Game complete</p><h1 id="game-title">Nice <em>eyes.</em></h1><p class="final-score">You scored <strong>{{ score }}</strong> points with a best streak of <strong>{{ bestStreak }}</strong>.</p><button class="next-button" @click="restartGame">Play again <span>↗</span></button></div>
    </section>
    <footer><span>Made for your inner color nerd</span><span>no pressure · just vibes</span></footer>
  </main>
</template>
