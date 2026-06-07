<script setup>
// no logic needed yet
</script>

<template>
  <div class="navbar">
    <div class="left">
      <h2>Sudoku</h2>
    </div>

    <div class="center">
      <slot name="center"></slot>
    </div>

    <div class="right">
      <slot name="right"></slot>
    </div>
  </div>
</template>

<style scoped>
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 60px;

  display: flex;
  align-items: center;

  box-sizing: border-box;
  padding: 0 16px;
  background: black;
  color: white;
  border-bottom: 1px solid #ddd;
}

/* 3 equal zones */
.left, .center, .right {
  flex: 1;
  display: flex;
  align-items: center;
  min-width: 0;
}

/* alignment inside zones */
.left {
  justify-content: flex-start;
}

.center {
  justify-content: center;
}

.right {
  justify-content: flex-end;
}
</style>