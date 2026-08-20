<script setup lang="ts">
import { computed, ref } from 'vue'

type ColorChoice = { hex: string; name: string }

const palette = [
  { name: 'coral', hex: '#ff7657' }, { name: 'cobalt', hex: '#4263eb' },
  { name: 'lemon', hex: '#f5d547' }, { name: 'mint', hex: '#43c6a3' },
  { name: 'plum', hex: '#9b5de5' }, { name: 'tangerine', hex: '#f59f45' },
  { name: 'sky', hex: '#56b4e9' }, { name: 'berry', hex: '#e65378' },
]
const target = ref<ColorChoice>(palette[0])
const choices = ref<ColorChoice[]>([])
const selected = ref<string | null>(null)
const score = ref(0)
const streak = ref(0)
const round = ref(1)
const isCorrect = computed(() => selected.value === target.value.hex)
const resultText = computed(() => isCorrect.value ? 'Perfect match!' : 'Close one!')

function shuffle<T>(items: T[]) { return [...items].sort(() => Math.random() - 0.5) }
function newRound() {
  const correct = palette[Math.floor(Math.random() * palette.length)]
  const wrong = shuffle(palette.filter((color) => color.hex !== correct.hex)).slice(0, 3)
  target.value = { hex: correct.hex, name: correct.name }
  choices.value = shuffle([{ hex: correct.hex, name: correct.name }, ...wrong])
  selected.value = null
}
function choose(choice: ColorChoice) {
  if (selected.value) return
  selected.value = choice.hex
  if (choice.hex === target.value.hex) { streak.value += 1; score.value += 10 + Math.max(0, streak.value - 1) * 2 } else streak.value = 0
}
function nextRound() { round.value += 1; newRound() }
newRound()
</script>

<template>
  <main class="game-shell">
    <nav class="topbar" aria-label="Main navigation">
      <a class="brand" href="./" aria-label="Hue Hunt home"><span class="brand-mark"><i></i><i></i><i></i></span><span>hue hunt</span></a>
      <span class="nav-note">a tiny color game</span>
    </nav>
    <section class="game-card" aria-labelledby="game-title">
      <div class="game-heading">
        <div><p class="eyebrow">Round {{ String(round).padStart(2, '0') }}</p><h1 id="game-title">Find the <em>match.</em></h1><p class="subtitle">One of these colors is the same as the target. Trust your eyes.</p></div>
        <div class="scoreboard" aria-label="Current score"><div><span>score</span><strong>{{ score }}</strong></div><div><span>streak</span><strong>{{ streak }}<small>×</small></strong></div></div>
      </div>
      <div class="target-area"><div class="target-swatch" :style="{ backgroundColor: target.hex }" aria-label="Target color"></div><div class="target-copy"><span class="target-label">target color</span><strong>What do you see?</strong><span class="target-hint">Pick the closest swatch below</span></div></div>
      <div class="choices" role="group" aria-label="Color choices">
        <button v-for="(choice, index) in choices" :key="choice.hex" class="color-choice" :class="{ correct: selected && choice.hex === target.hex, wrong: selected === choice.hex && choice.hex !== target.hex }" :style="{ '--choice-color': choice.hex }" :aria-label="`Color choice ${index + 1}`" :disabled="!!selected" @click="choose(choice)"><span class="choice-number">0{{ index + 1 }}</span><span v-if="selected && choice.hex === target.hex" class="choice-status">✓</span><span v-else-if="selected === choice.hex" class="choice-status">×</span></button>
      </div>
      <div class="result" :class="{ visible: !!selected, miss: !!selected && !isCorrect }" aria-live="polite"><div><span class="result-kicker">{{ isCorrect ? 'That’s it' : 'The answer was' }}</span><strong>{{ resultText }}</strong></div><button v-if="selected" class="next-button" @click="nextRound">Next round <span>→</span></button><span v-else class="waiting">Choose a color to begin</span></div>
    </section>
    <footer><span>Made for your inner color nerd</span><span>no pressure · just vibes</span></footer>
  </main>
</template>
