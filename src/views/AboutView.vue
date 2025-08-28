<script setup lang="ts">
import AboutPage from '../components/AboutPage.vue'
</script>

<template>
  <div class="about">
    <AboutPage />
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}
</style>
