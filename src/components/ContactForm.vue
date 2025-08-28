<script setup>
import { reactive, ref, computed } from 'vue'
import { initializeApp, getApps, getApp } from 'firebase/app'
import { getDatabase, ref as dbRef, push, serverTimestamp } from 'firebase/database'
import { initializeAppCheck, ReCaptchaV3Provider } from 'firebase/app-check'

// Firebase init with vite env
const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  databaseURL: import.meta.env.VITE_FIREBASE_DB_URL,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
}

const app = getApps().length ? getApp() : initializeApp(firebaseConfig)

// Optional App Check (recommended)
if (import.meta.env.VITE_RECAPTCHA_SITE_KEY) {
  try {
    initializeAppCheck(app, {
      provider: new ReCaptchaV3Provider(import.meta.env.VITE_RECAPTCHA_SITE_KEY),
      isTokenAutoRefreshEnabled: true,
    })
  } catch (_) {}
}

const db = getDatabase(app)

// --- Form state ---
const form = reactive({ name: '', email: '', message: '', website: '' })
const loading = ref(false)
const status = ref(null) // { type: 'success'|'error', message: string }

const emailRe = /^(?!.{255,})([\w.!#$%&'*+/=?`{|}~-]+)@([A-Za-z0-9-]+\.)+[A-Za-z]{2,}$/

const errors = computed(() => ({
  name: !form.name ? 'Please enter your name' : '',
  email: !form.email
    ? 'Please enter your email'
    : !emailRe.test(form.email)
      ? 'Enter a valid email'
      : '',
  message: !form.message ? 'Please enter a message' : '',
}))

function resetForm() {
  form.name = ''
  form.email = ''
  form.message = ''
  form.website = ''
}

async function onSubmit() {
  status.value = null
  if (errors.value.name || errors.value.email || errors.value.message) {
    status.value = { type: 'error', message: 'Please fix the errors above.' }
    return
  }
  // spam honeypot
  if (form.website) {
    status.value = { type: 'success', message: 'Thanks!' }
    resetForm()
    return
  }

  loading.value = true
  try {
    await push(dbRef(db, 'contacts/messages'), {
      name: form.name.slice(0, 200),
      email: form.email.slice(0, 320),
      message: form.message.slice(0, 5000),
      createdAt: serverTimestamp(),
      ua: navigator.userAgent,
    })

    status.value = { type: 'success', message: 'Thanks! Your message was sent.' }
    resetForm()
  } catch (err) {
    console.error(err)
    status.value = { type: 'error', message: 'Something went wrong. Please try again.' }
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <section class="contact">
    <form class="form" @submit.prevent="onSubmit" novalidate>
      <div class="field">
        <label for="name">Name</label>
        <input id="name" v-model.trim="form.name" type="text" required autocomplete="name" />
        <small v-if="errors.name">{{ errors.name }}</small>
      </div>

      <div class="field">
        <label for="email">Email</label>
        <input id="email" v-model.trim="form.email" type="email" required autocomplete="email" />
        <small v-if="errors.email">{{ errors.email }}</small>
      </div>

      <div class="field">
        <label for="message">Message</label>
        <textarea id="message" v-model.trim="form.message" rows="6" required></textarea>
        <small v-if="errors.message">{{ errors.message }}</small>
      </div>

      <input
        v-model="form.website"
        type="text"
        name="website"
        tabindex="-1"
        autocomplete="off"
        class="hp"
        aria-hidden="true"
      />

      <button type="submit" :disabled="loading">{{ loading ? 'Sending…' : 'Send' }}</button>

      <p v-if="status" :class="['status', status.type]">{{ status.message }}</p>
    </form>
  </section>
</template>

<style scoped>
.contact {
  max-width: 720px;
  margin: 0 auto;
  padding: 2rem 1rem;
}
.form {
  display: grid;
  gap: 1rem;
}
.field {
  display: grid;
  gap: 0.4rem;
}
label {
  font-weight: 600;
}
input,
textarea {
  width: 100%;
  padding: 0.7rem 0.8rem;
  border: 1px solid var(--color-border, #334155);
  border-radius: 0.6rem;
  background: var(--color-background-soft, #0b0b0b);
  color: var(--color-text, #e5e7eb);
}
small {
  color: #ef4444;
}
button {
  padding: 0.7rem 1rem;
  border-radius: 0.6rem;
  border: 1px solid #1f2937;
  background: #111827;
  color: #e5e7eb;
  cursor: pointer;
}
button[disabled] {
  opacity: 0.6;
  cursor: not-allowed;
}
.status {
  margin-top: 0.5rem;
}
.status.success {
  color: #10b981;
}
.status.error {
  color: #ef4444;
}
.hp {
  position: absolute;
  left: -9999px;
  opacity: 0;
}
</style>
