---
source_course: "vue-foundations"
source_lesson: "vue-foundations-lifecycle-hooks"
---

# Component Lifecycle Hooks

Every Vue component goes through a series of initialization steps—setting up data, compiling templates, mounting to the DOM, and updating when data changes. Along the way, Vue runs **lifecycle hooks** that let you execute code at specific stages.

## The Component Lifecycle

```
┌─────────────────────────────────────────┐
│              Component Created           │
│         (reactive data is set up)        │
└─────────────────────────────────────────┘
                    │
                    ▼
         ┌─────────────────────┐
         │    onBeforeMount    │
         │ (before DOM insert) │
         └─────────────────────┘
                    │
                    ▼
         ┌─────────────────────┐
         │      onMounted      │
         │   (DOM is ready)    │  ← Access DOM here
         └─────────────────────┘
                    │
           ┌───────┴───────┐
           │   Updates?    │
           └───────┬───────┘
                   │ yes
                   ▼
         ┌─────────────────────┐
         │   onBeforeUpdate    │
         │  (before DOM patch) │
         └─────────────────────┘
                    │
                    ▼
         ┌─────────────────────┐
         │      onUpdated      │
         │ (DOM is re-rendered)│
         └─────────────────────┘
                    │
           ┌───────┴───────┐
           │   Unmount?    │
           └───────┬───────┘
                   │ yes
                   ▼
         ┌─────────────────────┐
         │   onBeforeUnmount   │
         │  (still functional) │
         └─────────────────────┘
                    │
                    ▼
         ┌─────────────────────┐
         │     onUnmounted     │
         │   (cleanup time)    │  ← Cleanup here
         └─────────────────────┘
```

## onMounted - Most Common Hook

`onMounted()` runs after the component is added to the DOM. Use it for:

- Fetching initial data
- Setting up event listeners
- Integrating third-party libraries
- Accessing DOM elements

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const users = ref<User[]>([])
const isLoading = ref(true)
const error = ref<string | null>(null)

onMounted(async () => {
  try {
    const response = await fetch('/api/users')
    users.value = await response.json()
  } catch (e) {
    error.value = 'Failed to load users'
  } finally {
    isLoading.value = false
  }
})
</script>

<template>
  <div v-if="isLoading">Loading...</div>
  <div v-else-if="error">{{ error }}</div>
  <ul v-else>
    <li v-for="user in users" :key="user.id">{{ user.name }}</li>
  </ul>
</template>
```

## onUnmounted - Cleanup

`onUnmounted()` runs when the component is removed. Use it to:

- Remove event listeners
- Clear intervals/timeouts
- Disconnect from WebSockets
- Cancel pending requests

```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const mouseX = ref(0)
const mouseY = ref(0)

function handleMouseMove(event: MouseEvent) {
  mouseX.value = event.clientX
  mouseY.value = event.clientY
}

onMounted(() => {
  // Add event listener when mounted
  window.addEventListener('mousemove', handleMouseMove)
})

onUnmounted(() => {
  // Remove event listener when unmounted
  window.removeEventListener('mousemove', handleMouseMove)
})
</script>

<template>
  <p>Mouse position: {{ mouseX }}, {{ mouseY }}</p>
</template>
```

## onBeforeMount & onBeforeUnmount

These run just before mounting/unmounting:

```vue
<script setup lang="ts">
import { onBeforeMount, onBeforeUnmount } from 'vue'

onBeforeMount(() => {
  console.log('Component is about to be mounted')
  // DOM is not yet available
})

onBeforeUnmount(() => {
  console.log('Component is about to be unmounted')
  // Component is still fully functional
})
</script>
```

## onUpdated & onBeforeUpdate

These run when the component re-renders:

```vue
<script setup lang="ts">
import { ref, onUpdated, onBeforeUpdate } from 'vue'

const count = ref(0)

onBeforeUpdate(() => {
  console.log('About to update, count is still:', count.value)
})

onUpdated(() => {
  console.log('Updated! DOM now reflects:', count.value)
})
</script>

<template>
  <button @click="count++">Count: {{ count }}</button>
</template>
```

**Warning**: Don't mutate state inside `onUpdated`—it can cause infinite loops!

## Practical Example: Auto-Save

```vue
<script setup lang="ts">
import { ref, watch, onMounted, onUnmounted } from 'vue'

const content = ref('')
const lastSaved = ref<Date | null>(null)
const saveStatus = ref<'idle' | 'saving' | 'saved'>('idle')

let autoSaveInterval: number | null = null

async function save() {
  if (!content.value) return
  
  saveStatus.value = 'saving'
  try {
    await fetch('/api/document', {
      method: 'POST',
      body: JSON.stringify({ content: content.value })
    })
    lastSaved.value = new Date()
    saveStatus.value = 'saved'
  } catch (e) {
    console.error('Save failed:', e)
    saveStatus.value = 'idle'
  }
}

onMounted(() => {
  // Load existing content
  fetch('/api/document')
    .then(res => res.json())
    .then(data => {
      content.value = data.content || ''
    })
  
  // Set up auto-save every 30 seconds
  autoSaveInterval = window.setInterval(save, 30000)
})

onUnmounted(() => {
  // Clean up interval
  if (autoSaveInterval) {
    clearInterval(autoSaveInterval)
  }
  
  // Save on unmount
  save()
})

// Also save when content changes (debounced by watching)
let saveTimeout: number | null = null
watch(content, () => {
  saveStatus.value = 'idle'
  if (saveTimeout) clearTimeout(saveTimeout)
  saveTimeout = window.setTimeout(save, 2000)
})
</script>

<template>
  <div class="editor">
    <div class="toolbar">
      <span v-if="saveStatus === 'saving'">Saving...</span>
      <span v-else-if="saveStatus === 'saved'">
        Saved at {{ lastSaved?.toLocaleTimeString() }}
      </span>
      <button @click="save">Save Now</button>
    </div>
    <textarea v-model="content" placeholder="Start writing..."></textarea>
  </div>
</template>
```

## All Lifecycle Hooks Summary

| Hook | When it runs | Common uses |
|------|--------------|-------------|
| `onBeforeMount` | Before DOM insertion | Pre-mount setup |
| `onMounted` | After DOM insertion | Fetch data, DOM access |
| `onBeforeUpdate` | Before re-render | Read DOM before update |
| `onUpdated` | After re-render | DOM-dependent operations |
| `onBeforeUnmount` | Before removal | Last-minute cleanup |
| `onUnmounted` | After removal | Remove listeners, cleanup |

## Resources

- [Lifecycle Hooks](https://vuejs.org/guide/essentials/lifecycle.html) — Official Vue documentation on component lifecycle

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*