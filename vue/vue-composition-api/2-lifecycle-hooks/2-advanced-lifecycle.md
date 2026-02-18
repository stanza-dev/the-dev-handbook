---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-advanced-lifecycle"
---

# Advanced Lifecycle Patterns

Let's explore specialized lifecycle hooks and advanced patterns.

## onErrorCaptured

Capture errors from descendant components:

```vue
<script setup>
import { ref, onErrorCaptured } from 'vue'

const error = ref<Error | null>(null)

onErrorCaptured((err, instance, info) => {
  error.value = err
  console.error('Error captured:', err)
  console.log('Component:', instance)
  console.log('Error info:', info)  // 'setup function', 'render function', etc.
  
  // Return false to stop error propagation
  return false
})
</script>

<template>
  <div v-if="error" class="error">
    Something went wrong: {{ error.message }}
  </div>
  <slot v-else />
</template>
```

### Error Boundary Component

```vue
<!-- ErrorBoundary.vue -->
<script setup>
import { ref, onErrorCaptured } from 'vue'

const error = ref<Error | null>(null)
const errorInfo = ref('')

function reset() {
  error.value = null
  errorInfo.value = ''
}

onErrorCaptured((err, instance, info) => {
  error.value = err
  errorInfo.value = info
  return false  // Don't propagate
})
</script>

<template>
  <div v-if="error" class="error-boundary">
    <h2>Something went wrong</h2>
    <p>{{ error.message }}</p>
    <button @click="reset">Try Again</button>
  </div>
  <slot v-else />
</template>

<!-- Usage -->
<template>
  <ErrorBoundary>
    <RiskyComponent />
  </ErrorBoundary>
</template>
```

## onRenderTracked / onRenderTriggered (Dev Only)

Debug what causes re-renders:

```vue
<script setup>
import { ref, onRenderTracked, onRenderTriggered } from 'vue'

const count = ref(0)
const name = ref('Vue')

// Called when a reactive dependency is tracked during render
onRenderTracked((event) => {
  console.log('Tracked:', event)
  // { effect, target, type, key }
})

// Called when a tracked dependency triggers a re-render
onRenderTriggered((event) => {
  console.log('Triggered by:', event)
  debugger  // Pause here to inspect
})
</script>
```

## onActivated / onDeactivated (Keep-Alive)

For components inside `<KeepAlive>`:

```vue
<script setup>
import { ref, onMounted, onActivated, onDeactivated } from 'vue'

const visits = ref(0)

onMounted(() => {
  console.log('Mounted - only once')
})

onActivated(() => {
  visits.value++
  console.log('Activated - every time shown')
  // Resume animations, timers, etc.
})

onDeactivated(() => {
  console.log('Deactivated - hidden but not destroyed')
  // Pause animations, timers, etc.
})
</script>

<!-- Parent -->
<template>
  <KeepAlive>
    <component :is="currentTab" />
  </KeepAlive>
</template>
```

## onServerPrefetch (SSR)

For server-side rendering data fetching:

```vue
<script setup>
import { ref, onServerPrefetch, onMounted } from 'vue'

const data = ref(null)

// Only runs on server during SSR
onServerPrefetch(async () => {
  data.value = await fetchData()
})

// Runs on client
onMounted(() => {
  if (!data.value) {
    // Fetch if not hydrated from server
    fetchData().then(d => data.value = d)
  }
})
</script>
```

## Lifecycle in Composables

Composables can register their own lifecycle hooks:

```typescript
// useWindowSize.ts
import { ref, onMounted, onUnmounted } from 'vue'

export function useWindowSize() {
  const width = ref(0)
  const height = ref(0)
  
  function update() {
    width.value = window.innerWidth
    height.value = window.innerHeight
  }
  
  onMounted(() => {
    update()
    window.addEventListener('resize', update)
  })
  
  onUnmounted(() => {
    window.removeEventListener('resize', update)
  })
  
  return { width, height }
}

// Usage in component
<script setup>
import { useWindowSize } from './useWindowSize'

const { width, height } = useWindowSize()
// Lifecycle hooks are automatically registered
</script>
```

## Execution Order

When parent and child both have hooks:

```
Parent setup()
Child setup()
Child onBeforeMount()
Parent onBeforeMount()
Child onMounted()         ← Child mounts first
Parent onMounted()        ← Parent mounts after children

// On unmount:
Parent onBeforeUnmount()  ← Parent starts unmount
Child onBeforeUnmount()
Child onUnmounted()       ← Child unmounts first
Parent onUnmounted()
```

## Lifecycle Timing Tips

| Hook | DOM Access | Reactivity | Use For |
|------|------------|------------|---------|
| setup | ❌ | ✅ | Initial state, computed |
| onBeforeMount | ❌ | ✅ | Pre-render logic |
| onMounted | ✅ | ✅ | DOM manipulation, fetch |
| onBeforeUpdate | ✅ (old) | ✅ | Pre-update logic |
| onUpdated | ✅ (new) | ✅ | Post-update DOM work |
| onBeforeUnmount | ✅ | ✅ | Start cleanup |
| onUnmounted | ❌ | ✅ | Final cleanup |

## Resources

- [Lifecycle Diagram](https://vuejs.org/guide/essentials/lifecycle.html#lifecycle-diagram) — Visual lifecycle diagram

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*