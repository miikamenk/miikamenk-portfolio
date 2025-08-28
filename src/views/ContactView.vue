<script setup lang="ts">
import ContactForm from '../components/ContactForm.vue'
</script>

<template>
  <div class="about">
    <ContactForm />
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
