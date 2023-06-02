<template>
  <div v-if="isMenu" class="menu">
    <h1 class="menu__title">
      <span class="menu__decor">ball</span>sort
    </h1>
    <button @click="isMenu = false">
      Play
    </button>
  </div>
  <TheGame v-else @game-end="isMenu = true" />
</template>

<script setup>
import { ref } from 'vue'
import TheGame from './components/TheGame.vue'

const isMenu = ref(true)
</script>

<style lang="scss">
.menu {
  display: flex;
  flex-direction: column;
  align-items: center;

  &__title {
    font-size: 3rem;
    font-weight: 300;
    text-transform: uppercase;
  }

  &__decor {
    text-decoration: underline;
  }
}
</style>

<style>
body {
  font-family: system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,Noto Sans,sans-serif,BlinkMacSystemFont,Helvetica,Arial,Apple Color Emoji,Segoe UI Emoji,Segoe UI Symbol;
  color: #e1e1e1;
  margin: 0;
  padding: 0;
  background-color: #101526;
}

button {
  background-color: white;
  color: black;
  padding: 0.6rem 1rem;
  font-size: 1.3rem;
  margin: 0 0.2rem;
  border: 2px solid lightgray;
  cursor: pointer;
  position: relative;
}
</style>