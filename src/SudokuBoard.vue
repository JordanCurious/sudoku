<script setup>
import {ref, onMounted, watch} from 'vue'

const board = ref([])
const userBoard = ref([])
const fixed = ref([])

//sending signal of solving the puzzle to the parent component
let emit = defineEmits(['solved'])

//check which cell is selected, which cell need to be highlighted
let selected = ref(null)
let highlights = ref([])

const props = defineProps({
  resetSignal: Number,
  difficulty: String
})

async function loadSudoku() {
  const savedBoard = localStorage.getItem("sudoku-board")
  const savedUserBoard = localStorage.getItem("sudoku-user")

  if (savedBoard) {
    board.value = JSON.parse(savedBoard)

    if (savedUserBoard) {
      userBoard.value = JSON.parse(savedUserBoard)
    } else {
      userBoard.value = JSON.parse(JSON.stringify(board.value))
    }

    fixed.value = board.value.map(row =>
      row.map(cell => cell !== 0)
    )

    return
  }

  try {
    const boardResponse = await fetch(
      `https://sugoku.onrender.com/board?difficulty=${props.difficulty}`
    )

    const boardData = await boardResponse.json()

    board.value = boardData.board
    userBoard.value = JSON.parse(JSON.stringify(boardData.board))

    fixed.value = board.value.map(row =>
      row.map(cell => cell !== 0)
    )

    localStorage.setItem(
      "sudoku-board",
      JSON.stringify(boardData.board)
    )

    localStorage.setItem(
      "sudoku-user",
      JSON.stringify(userBoard.value)
    )
  } catch (err) {
    console.warn(err)
  }
}

function highlight(r, c) {
  const value = userBoard.value[r][c]

  // only trigger highlight if it's a fixed number AND non-zero AND no conflict
  if (value !== 0 && !checkConflict(r, c)) {
    highlights.value = []

    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        if (userBoard.value[i][j] === value) {
          highlights.value.push(9 * i + j)
        }
      }
    }
  } else {
    highlights.value = []
  }
}

function clickSelect(r, c) {
  const index = 9 * r + c

  if (selected.value === index) {
    selected.value = null
    highlights.value = []
    return
  }

  select(r, c)
}

function select(r, c) {
  selected.value = 9 * r + c

  highlight(r, c)
}

//style check for each cell
function cellClass(r, c) {
  return {
    active: highlights.value.includes(9 * r + c) || selected.value === 9 * r + c,

    topOuterBorder: c === 0,
    bottomOuterBorder: c === 8,
    leftOuterBorder: r === 0,
    rightOuterBorder: r === 8,

    topBorder: c === 3 || c === 6,
    bottomBorder: c === 2 || c === 5,
    leftBorder: r === 3 || r === 6,
    rightBorder: r === 2 || r === 5,

    fixed: fixed.value[r][c],
    conflict: checkConflict(r, c)
  }
}

function block(idx) {
  let blocks = [[0, 0], [3, 0], [6, 0], [0, 3], [3, 3], [6, 3], [0, 6], [3, 6], [6, 6]]
    let [x, y] = blocks[idx]
    let cells = [[x, y], [x + 1, y], [x + 2, y], [x, y + 1], [x + 1, y + 1], [x + 2, y + 1], [x, y + 2], [x + 1, y + 2], [x + 2, y + 2]]
                .map(([r, c]) => [9 * r + c, userBoard.value[r][c]])
    return cells
}

function checkConflict(r, c) {
  let value = userBoard.value[r][c]
  let blockIdx = Math.floor(c / 3) * 3 + Math.floor(r / 3)
  let blockCells = block(blockIdx)

  if (value === 0) {
    return false
  }

  for (let i = 0; i < 9; ++i) {
    if (i !== c && userBoard.value[r][i] === value) {
      return true
    }
    
    if (i !== r && userBoard.value[i][c] === value ) {
      return true
    }
    
    if (9 * r + c !== blockCells[i][0] && blockCells[i][1] === value) {
      return true
    }
  }

    return false
}

//Keyboard signal
function handleKeydown(event) {
    const keysToPrevent = [
      "ArrowUp",
      "ArrowDown",
      "ArrowLeft",
      "ArrowRight",
      "Backspace"
    ]

    if (keysToPrevent.includes(event.key)) {
      event.preventDefault()
    }

    if (selected.value === null) return

    let r = Math.floor(selected.value / 9)
    let c = selected.value % 9

    if (event.key === "ArrowRight") {
      c = (c + 1) % 9
    }

    if (event.key === "ArrowLeft") {
      c = (c + 8) % 9
    }

    if (event.key === "ArrowUp") {
      r = (r + 8) % 9
    }

    if (event.key === "ArrowDown") {
      r = (r + 1) % 9
    }

    if (event.key.startsWith("Arrow")) {
      select(r, c)
      return
    }

    if (fixed.value[r][c]) {
      return
    }

    if (event.key === "Backspace") {
        userBoard.value[r][c] = 0
        highlight(r, c)
    }

    const num = Number(event.key)

    if (num >= 1 && num <= 9) {
        userBoard.value[r][c] = num
        
        highlight(r, c)

        if (solutionCheck()) {
            emit("solved")
        }
    }

    localStorage.setItem(
      "sudoku-user",
      JSON.stringify(userBoard.value)
    )

}

//check if the solution is correct
function solutionCheck() {
  let digits = [1, 2, 3, 4, 5, 6, 7, 8, 9]

  function rowCheck(row) {
    for (let d of digits) {
      if (!userBoard.value[row].includes(d)) {
        return false
      }
    }

    return true
  }

  function columnCheck(col) {
    let column = digits.map(d => userBoard.value[d - 1][col])
    for (let d of digits) {
      if (!column.includes(d)) {
        return false
      }
    }

    return true
  }

  function blockCheck(b) {
    let cells = block(b).map(c => c[1])

    for (let d of digits) {
      if (!cells.includes(d)) {
        return false
      }
    }

    return true
  }

  for (let i = 0; i < 9; ++i) {
    if (!rowCheck(i) || !columnCheck(i) || !blockCheck(i)) {
      return false
    }
  }
  return true
}

watch(
  () => props.resetSignal,
  () => {
    localStorage.removeItem("sudoku-board")
    localStorage.removeItem("sudoku-user")

    selected.value = null
    highlights.value = []

    loadSudoku()   // 🔥 reset logic
  }
)

onMounted(() => {
  console.log(fixed)
  loadSudoku()
})

</script>

<template>
    <div class="board" tabindex="0" @keydown="handleKeydown($event)">
        <div  v-for="(row, r) in userBoard" :key="r" class="row">
            <button v-for="(cell, c) in row" 
                    :key="`${r}-${c}`" 
                    class="cell" 
                    :class="cellClass(r, c)" 
                    @click="clickSelect(r, c)">{{ cell || '\u00a0' }}</button>
        </div>
    </div>
</template>

<style scoped>
.board {
  display: flex;
  flex-direction: column;
  justify-content: center;
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
  color: gray;

  border: none;
  box-shadow: inset 0 0 0 1px #aaa;
}

.cell:focus {
  outline: none;
}

.active {
  background-color: #ffeaa7;
}

.conflict {
  background-color: #e57373;
  color: white;
}

/**
Since we transpose the sudoku board, we need to swap the role of left and top border, and right and bottom border respectively
*/
.leftOuterBorder {
  border-top: 4px solid black;
}

.rightOuterBorder {
    border-bottom: 4px solid black;
}

.topOuterBorder {
    border-left: 4px solid black;
}

.bottomOuterBorder {
    border-right: 4px solid black;
}

.leftBorder {
    border-top: 2px solid black;
}

.rightBorder {
    border-bottom: 2px solid black;
}

.topBorder {
    border-left: 2px solid black;
}

.bottomBorder {
    border-right: 2px solid black;
}

.fixed {
    color: black;
}
</style>
