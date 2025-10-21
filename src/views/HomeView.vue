<script setup lang="ts">
import { ref } from 'vue'
import DOMPurify from 'dompurify'
import VimEmulator from '../components/VimEmulator.vue'

const vimRef = ref<InstanceType<typeof VimEmulator> | null>(null)
const renderedHtml = ref<string | null>(null)

// Called when the user presses "Render"
async function onRender() {
  // get buffer from child
  const raw = vimRef.value?.getText?.() ?? ''
  // sanitize `raw` using DOMPurify in production to avoid XSS.
  renderedHtml.value = DOMPurify.sanitize(raw)
}

// Go back to emulator view
function onEdit() {
  renderedHtml.value = null
}
</script>

<template>
  <div class="home">
    <div class="terminal-wrapper">
      <div class="vim-controls">
        <button class="render-btn" @click="onRender" v-if="!renderedHtml">Render</button>
        <button class="render-btn" @click="onEdit" v-else>Back</button>
      </div>

      <VimEmulator v-show="!renderedHtml" ref="vimRef" />
      <div
        v-show="renderedHtml"
        class="rendered-terminal"
        aria-live="polite"
        v-html="renderedHtml"
      ></div>
    </div>
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
.rendered-terminal {
  width: 85vw;
  height: 50vh;
  color: #d1fae5;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Courier New', monospace;
}

.terminal-wrapper {
  position: relative; /* makes .vim-controls absolute inside it */
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem; /* optional: spacing between button and terminal */
}

.vim-controls {
  position: absolute;
  bottom: 100%; /* place above terminal */
  left: 97%; /* horizontally centered */
  transform: translateX(-50%) translateY(-0.5rem); /* small lift */
  display: flex;
  gap: 0.5rem;
  pointer-events: none;
  z-index: 30;
}

.vim-controls .render-btn {
  pointer-events: auto;
}

.render-btn {
  pointer-events: auto; /* allow clicks on the button itself */
  background: rgba(17, 24, 39, 0.85); /* glassy dark */
  color: #e6ffef;
  border: 1px solid rgba(255, 255, 255, 0.06);
  padding: 0.45rem 0.7rem;
  border-radius: 0.6rem;
  font:
    13px ui-monospace,
    monospace;
  box-shadow: 0 6px 18px rgba(2, 6, 23, 0.5);
  cursor: pointer;
  transition:
    transform 160ms ease,
    filter 160ms ease,
    opacity 160ms;
  opacity: 0.95;
}
.render-btn:hover {
  transform: translateY(-3px);
  filter: brightness(1.08);
}
.render-btn:active {
  transform: translateY(-1px) scale(0.995);
}
</style>
