<script setup>
import {ref} from 'vue'
const keyboard = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
</script>

<template>
<div class="keyboard">
    <div v-for="(row, r) in keyboard" :key="r" class="row">
        <button v-for="d in row" :key="d" class="cell">{{ d }}</button>
    </div>
</div>
</template>

<style scoped>
.keyboard {
    display: flex;
    flex-direction: column;
}

.row {
  display: flex;
  flex-direction: row;
}

.cell {
  display: flex;
  justify-content: center;
  align-items: center;

  width: 50px;
  height: 50px;

  font-size: 30px;
  color: black;

  border: none;
  box-shadow: inset 0 0 0 1px #aaa;
}

.cell:focus {
  outline: none;
}
</style>