<script setup lang="ts">
import VimEmulator from '../components/VimEmulator.vue'
</script>

<template>
  <div class="home">
    <VimEmulator />
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .home {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}
</style>
