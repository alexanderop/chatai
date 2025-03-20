<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'

// Props: the Vue code to render and a theme toggle
const props = withDefaults(defineProps<{
  code: string
  theme?: 'light' | 'dark'
}>(), {
  theme: 'light',
})

// Ref for the iframe element
const iframeRef = ref<HTMLIFrameElement | null>(null)

// Function to update the iframe content
function updateIframe() {
  const iframe = iframeRef.value
  if (!iframe)
    return

  // Get or create the document
  const doc = iframe.contentDocument
  if (!doc)
    return

  // Only do a full rewrite if iframe is empty or first load
  if (!doc.body || !doc.getElementById('app')) {
    // Write the HTML structure to the iframe
    doc.open()
    doc.write(`
      <html>
        <head>
          <!-- Load Vue 3 from CDN -->
          <script src="https://unpkg.com/vue@3/dist/vue.global.js"><\/script>
          <style>
            body {
              font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
              padding: 20px;
              transition: background-color 0.3s, color 0.3s;
            }
            html.light {
              color-scheme: light;
            }
            html.light body {
              background-color: #ffffff;
              color: #333333;
            }
            html.dark {
              color-scheme: dark;
            }
            html.dark body {
              background-color: #1e1e1e;
              color: #ffffff;
            }
            button {
              padding: 6px 12px;
              border-radius: 4px;
              cursor: pointer;
              margin: 5px 0;
              transition: background-color 0.3s;
            }
            html.light button {
              background-color: #f0f0f0;
              border: 1px solid #ccc;
              color: #333;
            }
            html.dark button {
              background-color: #2d2d2d;
              border: 1px solid #444;
              color: #fff;
            }
          </style>
        </head>
        <body>
          <div id="app"></div>
          <script id="vue-code">
            ${props.code}
          <\/script>
        </body>
      </html>
    `)
    doc.close()
  }
  else {
    // Just update the theme class without rebuilding everything
    doc.documentElement.className = props.theme
  }
}

// Set up the iframe when the component mounts
onMounted(() => {
  updateIframe()
})

// Watch for changes to the code or theme and update appropriately
watch(() => props.code, (newCode) => {
  const iframe = iframeRef.value
  if (!iframe || !iframe.contentDocument)
    return

  // Update only the script content
  const scriptElement = iframe.contentDocument.getElementById('vue-code')
  if (scriptElement) {
    scriptElement.textContent = newCode

    // Re-execute the script
    const newScript = iframe.contentDocument.createElement('script')
    newScript.id = 'vue-code'
    newScript.textContent = newCode
    scriptElement.parentNode?.replaceChild(newScript, scriptElement)
  }
  else {
    // Fallback to full update if script element not found
    updateIframe()
  }
})

// Watch theme changes - only update the class without rewriting everything
watch(() => props.theme, (newTheme) => {
  const iframe = iframeRef.value
  if (!iframe || !iframe.contentDocument)
    return

  iframe.contentDocument.documentElement.className = newTheme
})
</script>

<template>
  <iframe
    ref="iframeRef"
    style="width: 100%; height: 300px; border: none; border-radius: 6px; box-shadow: 0 2px 8px rgba(0,0,0,0.1);"
    :class="theme"
  />
</template>

<style scoped>
iframe.light {
  background-color: #fff;
}
iframe.dark {
  background-color: #1e1e1e;
}
</style>
