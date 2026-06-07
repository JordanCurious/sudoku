<script setup>
import {onMounted, ref, watch} from 'vue'
import SudokuBoard from './SudokuBoard.vue';
import Navbar from './Navbar.vue';

let isSolved = ref(false);

function onSolved() {
  isSolved.value = true;
}

const resetSignal = ref(0)
const difficulty = ref('easy')

function newGame() {
  resetSignal.value ++
}

function easy() {
  difficulty.value = 'easy'
}

function medium() {
  difficulty.value = 'medium'
}

function hard() {
  difficulty.value = 'hard'
}

function random() {
  difficulty.value = 'random'
}

function loadDifficulty() {
  const savedDifficulty = localStorage.getItem("difficulty")
  if (savedDifficulty) {
    difficulty.value = savedDifficulty
  } else {
    difficulty.value = 'easy'
  }
}

watch(difficulty, (newVal) => {
  localStorage.setItem("difficulty", newVal)
})

onMounted(() => {
  loadDifficulty()
})

</script>

<template>
  <Navbar>
    <template #right>
      <button class="new-game" @click="newGame()">New Game</button>
    </template>
    <template #center>
      <button class="difficulty" @click="easy" :class="{selected: difficulty === 'easy'}">Easy</button>
      <button class="difficulty" @click="medium" :class="{selected: difficulty === 'medium'}">Medium</button>
      <button class="difficulty" @click="hard" :class="{selected: difficulty === 'hard'}">Hard</button>
      <button class="difficulty" @click="random" :class="{selected: difficulty === 'random'}">Random</button>
    </template>
  </Navbar>
  <h2 v-if="isSolved" id="win-msg">You solved it !</h2>
  <h2 v-else>An interesting puzzle</h2>

  <div class="container">
    <SudokuBoard @solved="onSolved" :reset-signal="resetSignal" :difficulty="difficulty"/>
  </div>
  
</template>

<style scoped>
  h2 {
    display: flex;
    justify-content: center;
    align-items: center;
    padding-top: 60px;
  }
  h2#win-msg {
    color: red;
  }

  h1 {
    text-align: center;
  }

  button.new-game {
    background: none;
    border: none;
    color: #4ea1ff;
    cursor: pointer;
    font: inherit;
  }

  button.difficulty {
    background: none;
    border: none;
    color: white;
    cursor: pointer;
    font: inherit;
  }

  button.difficulty.selected {
    color: orange;
  }

  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 40px;
  }
</style>
