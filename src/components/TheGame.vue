<template>
  <div class="game-info">
    <button @click="$emit('game-end')">
      ←
    </button>
    <button @click="startGame">
      Restart
    </button>
    <span class="game-info__text">
      Moves: {{ stepsCount }}
    </span>
  </div>
  <div class="the-game">
    <div
      v-for="(tube, tubeId) in tubes"
      class="tube"
      :class="{ 'tube--completed': completedTubes[tubeId] }"
      @click="selectTube(tubeId)"
    >
      <template v-for="ball in tube">
        <div
            class="ball"
            :style="styleByColor[ball]"
        />
      </template>

      <div
          v-if="selected.tube === tubeId"
          class="ball ball--selected"
          :style="styleByColor[selected.ball]"
      />
    </div>
  </div>

  <div v-if="gameIsCompleted" class="win-modal">
    <h2 class="win-modal__title">
      You won!
    </h2>
    <p class="win-modal__text">
      In {{ stepsCount }} moves
    </p>
    <button @click="startGame">
      New Game
    </button>
  </div>
</template>

<script setup>
import {ref, computed, watchEffect} from 'vue'

const props = defineProps({
  colorsSettings: {
    type: Object,
    default: () => ({
      green: {
        '--main-color': '#247516',
        '--light-color': '#70FF00',
      },
      yellow: {
        '--main-color': '#8F7E22',
        '--light-color': '#FFE600',
      },
      cyan: {
        '--main-color': '#29777C',
        '--light-color': '#00FFF0',
      },
      blue: {
        '--main-color': '#466799',
        '--light-color': '#00B2FF',
      },
    })
  },
  additionalTubes: {
    type: Number,
    default: 2
  },
  additionalSpace: {
    type: Number,
    default: 1
  }
})

const gameParams = computed(() => {
  const colors = Object.keys(props.colorsSettings)
  const colorsCount = colors.length

  return {
    colors,
    startCountOfBallsOfEachColor: colorsCount,
    startFilledTubes: colorsCount,
    startBallsInTube: colorsCount,
    tubeVolume: colorsCount + props.additionalSpace,
    tubesCount: colorsCount + props.additionalTubes
  }
})

const styleByColor = computed(() => props.colorsSettings)

const tubes = ref([])
const selected = ref({
  tube: -1,
  ball: undefined
})
const stepsCount = ref(0)

const completedTubes = computed(() => {
  return tubes.value.map((tube) =>
      (tube.length === gameParams.value.startBallsInTube) &&
      (new Set(tube).size === 1)
  )
})

const gameIsCompleted = computed(() => {
  const completedCount = completedTubes.value.reduce((acc, isComplete) => isComplete ? acc + 1 : acc, 0)
  return completedCount === gameParams.value.startFilledTubes
})

function clearSelected() {
  selected.value.tube = -1
  selected.value.ball = undefined
}

watchEffect(startGame)
function startGame() {
  clearSelected()
  stepsCount.value = 0

  const colorIter = gameParams.value.colors
      .map((color) => Array(gameParams.value.startCountOfBallsOfEachColor).fill(color))
      .flat()
      .sort(() => Math.random() - 0.5)
      [Symbol.iterator]()

  const createdTubes = new Array(gameParams.value.tubesCount)
  for (let tubeId = 0; tubeId < gameParams.value.startFilledTubes; tubeId += 1) {
    const tube = []
    for (let i = 0; i < gameParams.value.startFilledTubes; i += 1) {
      tube.push(colorIter.next().value)
    }
    createdTubes[tubeId] = tube
  }
  for (let tubeId = gameParams.value.startFilledTubes; tubeId < gameParams.value.tubesCount; tubeId += 1) {
    createdTubes[tubeId] = []
  }
  tubes.value = createdTubes
}

function selectTube(tubeId) {
  const currentTube = tubes.value[tubeId]
  const currentTubeLength = tubes.value[tubeId].length
  const selectedBall = selected.value.ball

  if (!selectedBall) {
    selected.value.tube = tubeId
    selected.value.ball = currentTube.pop()
    return
  }

  if (selected.value.tube === tubeId) {
    currentTube.push(selectedBall)
    clearSelected()
    return
  }

  if (currentTubeLength === gameParams.value.tubeVolume) {
    return
  }

  if (currentTubeLength > 0 && currentTube.at(-1) !== selectedBall) {
    return
  }

  currentTube.push(selectedBall)
  clearSelected()
  stepsCount.value += 1
}

</script>

<style lang="scss">
.game-info {
  display: flex;
  flex-direction: row;
  gap: 10px;
  align-items: center;
  justify-content: center;
  margin-top: 24px;

  &__text {
    font-size: 1.2em;
  }
}

.the-game {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 1.5em;
  padding: 32px 12px;

  --ball-size: 2em;
  --ball-space: calc(var(--ball-size) + 0.4em);
}

.tube {
  --tube-volume: v-bind("gameParams.tubeVolume");

  position: relative;

  display: flex;
  flex-direction: column-reverse;
  justify-content: flex-start;
  flex-shrink: 0;
  align-items: center;

  width: calc(var(--ball-space) + 0.4rem);
  height: calc((var(--tube-volume) - 0.7) * var(--ball-space));
  padding-bottom: 0.4rem;
  padding-top: 0.4rem;
  margin-top: calc(0.5rem + var(--ball-space));

  border: 2px solid lightgray;
  border-top-width: 4px;
  border-bottom-left-radius: var(--ball-space);
  border-bottom-right-radius: var(--ball-space);

  &--completed {
    background: lightgray;
  }
}

.ball {
  width: var(--ball-size);
  height: var(--ball-size);
  border-radius: 50%;
  border: 2px solid black;
  margin: 1px;
  flex-shrink: 0;
  background: radial-gradient( circle at 65% 15%, white 1px, var(--light-color) 3%, var(--main-color) 60%, var(--light-color) 100% );
  position: relative;

  &--selected {
    position: absolute;
    top: calc(-1 * (var(--ball-space) + 0.4rem));
  }
}

.win-modal {
  display: flex;
  flex-flow: column;
  position: fixed;
  inset: 0;
  background-color: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(6px);
  align-items: center;
  padding-top: 5rem;

  &__title {
    margin: 0;
    font-size: 3em;
    color: black;
    text-shadow: white 0 0 2px;
  }

  &__text {
    font-size: 2em;
    font-weight: 500;
    color: black;
    text-shadow: white 0 0 2px;
  }
}
</style>