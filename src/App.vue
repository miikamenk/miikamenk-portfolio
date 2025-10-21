<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import { ref } from 'vue'

const isMenuOpen = ref(false)

function toggleMenu() {
  isMenuOpen.value = !isMenuOpen.value
}
</script>

<template>
  <header>
    <div class="header-container">
      <div class="left-section">
        <img alt="Vue logo" class="logo" src="@/assets/logo.png" width="150" height="150" />
        <h1 class="port-header">miikamenk</h1>
      </div>

      <nav class="desktop-nav">
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/projects">Projects</RouterLink>
        <RouterLink to="/contact">Contact</RouterLink>
      </nav>

      <div class="mobile-nav">
        <button @click="toggleMenu" class="menu-button">☰</button>
        <div v-if="isMenuOpen" class="dropdown-menu">
          <RouterLink to="/" @click="toggleMenu">Home</RouterLink>
          <RouterLink to="/projects" @click="toggleMenu">Projects</RouterLink>
          <RouterLink to="/contact" @click="toggleMenu">Contact</RouterLink>
        </div>
      </div>
    </div>
  </header>

  <div class="content">
    <RouterView />
  </div>
</template>

<style scoped>
header {
  width: 100%;
  position: fixed;
  top: 0;
  left: 0;
  background: var(--color-background);
  z-index: 1000;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.content {
  margin: 0 auto;
  padding: 2rem;
  padding-top: calc(2rem + 80px);
  box-sizing: border-box;
}

.header-container {
  margin: 0 auto;
  padding: 0.75rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.left-section {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.logo {
  width: 100px;
  height: 100px;
}

.port-header {
  font-size: 2.5rem;
  font-weight: 900;
  margin: 0;
  color: var(--color-text);
}

.desktop-nav {
  display: flex;
  gap: 2rem;
}

.desktop-nav a {
  text-decoration: none;
  color: var(--color-text);
  font-size: 1.05rem;
  padding: 0.25rem 0.5rem;
  position: relative;
  transition: color 0.2s ease;
}

.desktop-nav a::after {
  content: '';
  position: absolute;
  bottom: -4px;
  left: 0;
  height: 2px;
  width: 0%;
  background: var(--color-text);
  transition: width 0.25s ease;
}

.desktop-nav a:hover::after,
.desktop-nav a.router-link-exact-active::after {
  width: 100%;
}

.mobile-nav {
  display: none;
  position: relative;
}

.menu-button {
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: var(--color-text);
}

.dropdown-menu {
  position: absolute;
  top: 110%;
  right: 0;
  background: var(--color-background);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  min-width: 150px;
  animation: fadeIn 0.2s ease;
}

.dropdown-menu a {
  text-decoration: none;
  color: var(--color-text);
  padding: 0.5rem;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}

.dropdown-menu a:hover,
.dropdown-menu a.router-link-exact-active {
  background-color: var(--color-border-hover);
}

@media (max-width: 768px) {
  .desktop-nav {
    display: none;
  }
  .mobile-nav {
    display: block;
  }
}

@media (min-width: 769px) {
  .mobile-nav {
    display: none;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-5px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
