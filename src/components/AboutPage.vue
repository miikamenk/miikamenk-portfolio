<script setup lang="ts"></script>

<template>
  <section class="about-page">
    <div class="card">
      <h1 class="name">miikamenk</h1>
      <h2 class="subtitle">Self-taught software developer</h2>

      <h3 class="section-title">About me</h3>
      <ul class="about-list">
        <li>Has been writing software for over a decade</li>
        <li>Hobbyist in electronics design and custom keyboards</li>
        <li>Open source advocator</li>
        <li>Vim lover</li>
      </ul>
    </div>
  </section>
</template>

<style scoped>
.about-page {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 80vh;
  padding: 2rem;
}

.card {
  background: var(--color-background);
  padding: 2rem 3rem;
  border-radius: 1rem;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
  max-width: 700px;
  width: 100%;
  text-align: center;
  animation: fadeInUp 0.6s ease;
}

.name {
  font-size: 2.5rem;
  font-weight: 800;
  margin-bottom: 0.5rem;
  color: var(--color-text);
}

.subtitle {
  font-size: 1.2rem;
  font-weight: 500;
  margin-bottom: 2rem;
  color: var(--color-text-secondary, #666);
}

.section-title {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 1rem;
}

.about-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.about-list li {
  font-size: 1.05rem;
  margin: 0.5rem 0;
  position: relative;
  padding-left: 1.5rem;
  text-align: left;
}

.about-list li::before {
  content: '▹';
  position: absolute;
  left: 0;
  color: var(--color-accent, #42b883);
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
