---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-lifecycle-overview"
---

# Component Lifecycle in Composition API

Lifecycle hooks let you run code at specific points in a component's life. The Composition API uses `on`-prefixed functions.

## Lifecycle Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        CREATION                              │
├─────────────────────────────────────────────────────────────┤
│  setup() runs                                                │
│  └─ Reactive state created                                   │
│  └─ Computed/watchers set up                                 │
│  └─ onBeforeMount callbacks registered                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        MOUNTING                              │
├─────────────────────────────────────────────────────────────┤
│  onBeforeMount()  ← DOM not yet created                      │
│       ↓                                                      │
│  [DOM RENDERED]                                              │
│       ↓                                                      │
│  onMounted()      ← DOM is ready, refs populated             │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        UPDATING                              │
├─────────────────────────────────────────────────────────────┤
│  [Reactive state changes]                                    │
│       ↓                                                      │
│  onBeforeUpdate() ← Before DOM patch                         │
│       ↓                                                      │
│  [DOM PATCHED]                                               │
│       ↓                                                      │
│  onUpdated()      ← DOM updated                              │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      UNMOUNTING                              │
├─────────────────────────────────────────────────────────────┤
│  onBeforeUnmount() ← Component still functional              │
│       ↓                                                      │
│  [DESTROYED]                                                 │
│       ↓                                                      │
│  onUnmounted()     ← Cleanup complete                        │
└─────────────────────────────────────────────────────────────┘
```

## Basic Usage

```vue
<script setup>
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  ref
} from 'vue'

const count = ref(0)
const element = ref<HTMLElement | null>(null)

// Before DOM is created
onBeforeMount(() => {
  console.log('Component is about to mount')
  console.log(element.value)  // null - DOM doesn't exist yet
})

// DOM is ready
onMounted(() => {
  console.log('Component mounted!')
  console.log(element.value)  // <div>...</div>
  
  // Safe to:
  // - Access DOM elements
  // - Start timers/intervals
  // - Add event listeners
  // - Fetch data
})

// Before DOM updates
onBeforeUpdate(() => {
  console.log('About to update, count is:', count.value)
})

// After DOM updates
onUpdated(() => {
  console.log('DOM updated with count:', count.value)
})

// Before component is destroyed
onBeforeUnmount(() => {
  console.log('About to unmount')
  // Still have access to DOM and state
})

// After component is destroyed
onUnmounted(() => {
  console.log('Component unmounted')
  // Cleanup here:
  // - Remove event listeners
  // - Clear timers
  // - Cancel subscriptions
})
</script>

<template>
  <div ref="element">
    <button @click="count++">Count: {{ count }}</button>
  </div>
</template>
```

## Common Patterns

### Fetching Data on Mount

```vue
<script setup>
import { ref, onMounted } from 'vue'

const data = ref(null)
const loading = ref(true)
const error = ref(null)

onMounted(async () => {
  try {
    const response = await fetch('/api/data')
    data.value = await response.json()
  } catch (e) {
    error.value = e
  } finally {
    loading.value = false
  }
})
</script>
```

### Event Listener Cleanup

```vue
<script setup>
import { onMounted, onUnmounted } from 'vue'

function handleResize() {
  console.log('Window resized:', window.innerWidth)
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>
```

### Timer Cleanup

```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const time = ref(new Date())
let intervalId: number

onMounted(() => {
  intervalId = setInterval(() => {
    time.value = new Date()
  }, 1000)
})

onUnmounted(() => {
  clearInterval(intervalId)
})
</script>
```

### Third-Party Library Integration

```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import Chart from 'chart.js/auto'

const chartRef = ref<HTMLCanvasElement | null>(null)
let chartInstance: Chart | null = null

onMounted(() => {
  if (chartRef.value) {
    chartInstance = new Chart(chartRef.value, {
      type: 'bar',
      data: { /* ... */ }
    })
  }
})

onUnmounted(() => {
  chartInstance?.destroy()
})
</script>

<template>
  <canvas ref="chartRef"></canvas>
</template>
```

## Multiple Hooks of Same Type

You can register multiple hooks of the same type:

```vue
<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  console.log('First onMounted')
})

onMounted(() => {
  console.log('Second onMounted')
})

// Both run in order
</script>
```

This is useful when composables need their own lifecycle hooks.

## Resources

- [Lifecycle Hooks](https://vuejs.org/guide/essentials/lifecycle.html) — Official documentation on component lifecycle

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*