<script setup>
import { onMounted, onBeforeUnmount, ref, computed } from 'vue'

//config
const MESSAGE = `Hi, I'm miikamenk
Software dev mostly focused on backend.
I like building custom keyboards, coding, linux and vim.
Try out this interactive vim terminal I made and check out the rest of my website.`
const TYPE_DELAY = 40 // ms/char
const PLACEHOLDER_LINES = 8 // leading tildes like Vim

// state controls
const text = ref('') // visible buffer
const cursor = ref(0) // index in text
const typingDone = ref(false)
const insertMode = ref(false)

const pendingOp = ref(null) // 'd' | null
let opBuffer = '' // e.g. 'i' for 'diw'
let desiredCol = null // preferred column when moving vertically
let timer = null

// derived
const beforeText = computed(() => text.value.slice(0, cursor.value))
const afterText = computed(() => text.value.slice(cursor.value))

const lineCol = computed(() => {
  const { line, col } = getLineCol(cursor.value)
  // Vim shows 1-based
  return { line: line + 1, col: col + 1 }
})

const statusText = computed(() => {
  if (insertMode.value) return '-- INSERT --'
  if (pendingOp.value === 'd') return 'd …'
  return 'NORMAL'
})

// typing animation bullshit
function startTyping() {
  let i = 0
  timer = setInterval(() => {
    if (i >= MESSAGE.length) return stopTyping(true)
    insertChars(MESSAGE[i++])
    moveCursorTo(text.value.length)
  }, TYPE_DELAY)
}

function stopTyping(done = false) {
  if (timer) {
    clearInterval(timer)
    timer = null
  }
  if (done) typingDone.value = true
}

function skipTyping() {
  if (typingDone.value) return
  text.value = MESSAGE
  typingDone.value = true
  stopTyping()
  moveCursorTo(text.value.length)
}

// text geometry for navigation
function getLineStart(idx) {
  return text.value.lastIndexOf('\n', Math.max(0, idx - 1)) + 1
}

function getLineEnd(idx) {
  const pos = text.value.indexOf('\n', idx)
  return pos === -1 ? text.value.length : pos
}

function getLineCol(idx) {
  const start = getLineStart(idx)
  const line = (text.value.slice(0, idx).match(/\n/g) || []).length
  const col = idx - start
  return { line, col }
}

function updateDesiredColFromCursor() {
  desiredCol = getLineCol(cursor.value).col
}

function clamp(n, min, max) {
  return Math.max(min, Math.min(max, n))
}

function moveCursorTo(idx) {
  cursor.value = clamp(idx, 0, text.value.length)
  updateDesiredColFromCursor()
}

// navigation
function moveLeft() {
  if (cursor.value > 0) moveCursorTo(cursor.value - 1)
}
function moveRight() {
  if (cursor.value < text.value.length) moveCursorTo(cursor.value + 1)
}

function moveUp() {
  const start = getLineStart(cursor.value)
  if (start === 0) return
  const prevEnd = start - 1
  const prevStart = text.value.lastIndexOf('\n', prevEnd - 1) + 1
  const prevLen = prevEnd - prevStart
  const col = desiredCol ?? getLineCol(cursor.value).col
  moveCursorTo(prevStart + Math.min(col, prevLen))
}

function moveDown() {
  const end = getLineEnd(cursor.value)
  if (end === text.value.length) return
  const nextStart = end + 1
  const nextEnd = getLineEnd(nextStart)
  const nextLen = nextEnd - nextStart
  const col = desiredCol ?? getLineCol(cursor.value).col
  moveCursorTo(nextStart + Math.min(col, nextLen))
}

// editing primitives
function setText(sliceStart, sliceEnd, insert = '') {
  text.value = text.value.slice(0, sliceStart) + insert + text.value.slice(sliceEnd)
}

function insertChars(ch) {
  setText(cursor.value, cursor.value, ch)
  moveCursorTo(cursor.value + ch.length)
}

function backspaceChar() {
  if (cursor.value === 0) return
  setText(cursor.value - 1, cursor.value)
  moveCursorTo(cursor.value - 0) // stay at same visual col
}

function deleteCharUnderCursor() {
  if (cursor.value >= text.value.length) return
  setText(cursor.value, cursor.value + 1)
  moveCursorTo(cursor.value) // clamp handled in moveCursorTo
}

function deleteCurrentLine() {
  const start = getLineStart(cursor.value)
  const end = getLineEnd(cursor.value)
  const cutTo = end < text.value.length ? end + 1 : end // include newline if not last line
  setText(start, cutTo)
  moveCursorTo(Math.min(start, text.value.length))
}

// word utils
const isWordChar = (c) => /[A-Za-z0-9_]/.test(c)

function deleteInnerWord() {
  const t = text.value
  if (!t.length) return

  let i = cursor.value
  // If on whitespace, prefer word to the right; else use word under/left
  if (!isWordChar(t[i]) && i > 0 && isWordChar(t[i - 1])) i--

  // Find a word around i (skip to next word if on whitespace)
  if (!isWordChar(t[i])) {
    let k = i
    while (k < t.length && !isWordChar(t[k])) k++
    if (k >= t.length || !isWordChar(t[k])) return
    i = k
  }

  let left = i
  let right = i
  while (left > 0 && isWordChar(t[left - 1])) left--
  while (right < t.length && isWordChar(t[right])) right++

  setText(left, right)
  moveCursorTo(left)
}

// operator pending for exampel diw ddg
function enterOp(op) {
  pendingOp.value = op
  opBuffer = ''
}

function handlePendingOp(key) {
  if (pendingOp.value !== 'd') return false

  // dd
  if (key === 'd') {
    deleteCurrentLine()
    pendingOp.value = null
    opBuffer = ''
    return true
  }

  // diw
  if (key === 'i' && opBuffer === '') {
    opBuffer = 'i'
    return true
  }
  if (key === 'w' && opBuffer === 'i') {
    deleteInnerWord()
    pendingOp.value = null
    opBuffer = ''
    return true
  }

  // Unknown sequence → cancel
  pendingOp.value = null
  opBuffer = ''
  return true
}

// vim moodes
function enterInsertHere() {
  insertMode.value = true
  typingDone.value = true
  stopTyping()
}

function toLineStartInsert() {
  moveCursorTo(getLineStart(cursor.value))
  enterInsertHere()
}

function appendInsert() {
  if (cursor.value < text.value.length) moveCursorTo(cursor.value + 1)
  enterInsertHere()
}

// handling keys
function prevent(e, fn) {
  fn()
  e.preventDefault()
}

function onKeydown(e) {
  // Skip typing for any editing/nav (except space/enter which already skip below)
  if (!typingDone.value && ![' ', 'Enter'].includes(e.key)) skipTyping()

  // Insert mode
  if (insertMode.value) {
    const key = e.key
    if (key === 'Escape') return prevent(e, () => (insertMode.value = false))
    if (key === 'Backspace') return prevent(e, backspaceChar)
    if (key === 'Enter') return prevent(e, () => insertChars('\n'))
    if (key === 'Tab') return prevent(e, () => insertChars('  '))
    if (key === 'ArrowLeft') return prevent(e, moveLeft)
    if (key === 'ArrowRight') return prevent(e, moveRight)
    if (key === 'ArrowUp') return prevent(e, moveUp)
    if (key === 'ArrowDown') return prevent(e, moveDown)
    if (key.length === 1 && !e.ctrlKey && !e.metaKey) return prevent(e, () => insertChars(key))
    return
  }

  // Normal mode
  if (pendingOp.value) {
    if (handlePendingOp(e.key)) return e.preventDefault()
  }

  const key = e.key

  // Mode switches
  if (key === 'i') return prevent(e, enterInsertHere)
  if (key === 'I') return prevent(e, toLineStartInsert)
  if (key === 'a') return prevent(e, appendInsert)

  // Movement
  if (key === 'h' || key === 'ArrowLeft') return prevent(e, moveLeft)
  if (key === 'l' || key === 'ArrowRight') return prevent(e, moveRight)
  if (key === 'k' || key === 'ArrowUp') return prevent(e, moveUp)
  if (key === 'j' || key === 'ArrowDown') return prevent(e, moveDown)

  // Editing
  if (key === 'x') return prevent(e, deleteCharUnderCursor)
  if (key === 'd') return prevent(e, () => enterOp('d'))

  // Finish typing on space/enter
  if (!typingDone.value && (key === ' ' || key === 'Enter')) {
    return prevent(e, skipTyping)
  }
}

onMounted(() => {
  window.addEventListener('keydown', onKeydown)
  startTyping()
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKeydown)
  stopTyping()
})
</script>

<template>
  <section class="vim-wrap">
    <div class="vim-window" @click="skipTyping">
      <div class="vim-title">— VIM —</div>

      <div class="vim-buffer" aria-live="polite" aria-atomic="true">
        <pre class="vim-line" v-for="n in PLACEHOLDER_LINES" :key="n">~</pre>
        <pre class="vim-text">
<span>{{ beforeText }}</span><span class="cursor" :class="{ off: !insertMode && typingDone }">█</span><span>{{ afterText }}</span>
        </pre>
      </div>

      <div class="vim-status">
        <span>"Home" [No Name]</span>
        <span class="mode" :class="{ insert: insertMode }">{{ statusText }}</span>
        <span class="pos">{{ lineCol.line }},{{ lineCol.col }}</span>
      </div>
    </div>

    <div class="hint">
      <p>
        Movement: <kbd>h</kbd><kbd>j</kbd><kbd>k</kbd><kbd>l</kbd>, <br /><kbd>d d</kbd> (delete
        line), <br /><kbd>d i w</kbd> (delete inner word), <br /><kbd>x</kbd> (delete char),
        <br /><kbd>i</kbd>/<kbd>I</kbd> (insert / start of line), <kbd>a</kbd> (append), <br /><kbd
          >Esc</kbd
        >
        to exit insert. <kbd>Backspace</kbd> deletes, <kbd>Enter</kbd> new line.
      </p>
      <router-link to="/projects" class="btn">View Projects</router-link>
    </div>
  </section>
</template>

<style scoped>
/* style was made mostly by chatgpt */
.vim-wrap {
  display: grid;
  gap: 1rem;
  place-items: center;
  padding: 4rem 1rem;
}
.vim-window {
  width: min(900px, 95vw);
  border-radius: 0.75rem;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
  background: #000;
  border: 1px solid #1f2937;
}
.vim-title {
  background: #0b0b0b;
  color: #9ca3af;
  font:
    500 12px ui-monospace,
    SFMono-Regular,
    Menlo,
    Monaco,
    Consolas,
    'Liberation Mono',
    'Courier New',
    monospace;
  padding: 0.5rem 0.75rem;
  letter-spacing: 0.08em;
  border-bottom: 1px solid #1f2937;
}
.vim-buffer {
  padding: 1rem 0.75rem 1.25rem;
  min-height: 220px;
  color: #a7f3d0;
  background: #000;
}
.vim-line,
.vim-text {
  margin: 0;
  white-space: pre-wrap;
  font:
    16px/1.6 ui-monospace,
    SFMono-Regular,
    Menlo,
    Monaco,
    Consolas,
    'Liberation Mono',
    'Courier New',
    monospace;
}
.vim-line {
  color: #374151;
}
.vim-text {
  color: #d1fae5;
}
.cursor {
  display: inline-block;
  width: 0.6ch;
  animation: blink 1s steps(1) infinite;
}
.cursor.off {
  animation: none;
  opacity: 0.6;
}
@keyframes blink {
  50% {
    opacity: 0;
  }
}
.vim-status {
  display: grid;
  grid-template-columns: 1fr auto auto;
  gap: 0.75rem;
  align-items: center;
  padding: 0.4rem 0.75rem;
  color: #e5e7eb;
  background: #111827;
  font:
    12px ui-monospace,
    SFMono-Regular,
    Menlo,
    Monaco,
    Consolas,
    'Liberation Mono',
    'Courier New',
    monospace;
  border-top: 1px solid #1f2937;
}
.vim-status .mode.insert {
  color: #10b981;
}
.vim-status .pos {
  color: #9ca3af;
}
.hint {
  text-align: center;
}
.hint .btn {
  display: inline-block;
  margin-top: 0.5rem;
  padding: 0.6rem 1rem;
  border-radius: 0.75rem;
  background: #111827;
  color: #e5e7eb;
  text-decoration: none;
  border: 1px solid #1f2937;
}
.hint .btn:hover {
  filter: brightness(1.1);
}
@media (prefers-color-scheme: light) {
  .vim-window {
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  }
}
</style>
