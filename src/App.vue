<script setup lang="ts">
import { ref } from 'vue'

// Sample Vue code to render in the iframe
const sampleCode = ref(`
  const { createApp } = Vue
  const App = {
    template: '<div>Hello from the Vue Playground! <button @click="count++">{{ count }}</button></div>',
    data() {
      return { count: 0 }
    }
  }
  createApp(App).mount('#app')
`)

const theme = ref<'light' | 'dark'>('light')
</script>

<template>
  <div>
    <button @click="theme = theme === 'light' ? 'dark' : 'light'">
      Toggle Theme (Current: {{ theme }})
    </button>
    <Sandbox :code="sampleCode" :theme="theme" />
  </div>
</template>
