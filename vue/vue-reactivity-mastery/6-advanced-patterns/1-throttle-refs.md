---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-debounce-throttle-refs"
---

# Debounced and Throttled Refs

Control how often reactive updates propagate for performance and UX.

## Debounced Ref

Delay updates until input stops:

```typescript
import { ref, watch, type Ref } from 'vue'

export function useDebouncedRef<T>(value: T, delay = 300): Ref<T> {
  const debouncedValue = ref(value) as Ref<T>
  let timeout: ReturnType<typeof setTimeout>
  
  watch(
    () => value,
    (newValue) => {
      clearTimeout(timeout)
      timeout = setTimeout(() => {
        debouncedValue.value = newValue
      }, delay)
    }
  )
  
  return debouncedValue
}

// Usage
const searchInput = ref('')
const debouncedSearch = useDebouncedRef(searchInput.value, 500)
```

### Custom Debounced Ref with customRef

```typescript
import { customRef } from 'vue'

export function useDebouncedRef<T>(initialValue: T, delay = 300) {
  let timeout: ReturnType<typeof setTimeout>
  let value = initialValue
  
  return customRef((track, trigger) => ({
    get() {
      track()
      return value
    },
    set(newValue: T) {
      clearTimeout(timeout)
      timeout = setTimeout(() => {
        value = newValue
        trigger()
      }, delay)
    }
  }))
}
```

## Throttled Ref

Limit updates to a maximum frequency:

```typescript
import { customRef } from 'vue'

export function useThrottledRef<T>(initialValue: T, interval = 100) {
  let value = initialValue
  let lastUpdate = 0
  let pendingValue: T | null = null
  let timeout: ReturnType<typeof setTimeout> | null = null
  
  return customRef((track, trigger) => ({
    get() {
      track()
      return value
    },
    set(newValue: T) {
      const now = Date.now()
      
      if (now - lastUpdate >= interval) {
        value = newValue
        lastUpdate = now
        trigger()
      } else {
        pendingValue = newValue
        if (!timeout) {
          timeout = setTimeout(() => {
            if (pendingValue !== null) {
              value = pendingValue
              pendingValue = null
              lastUpdate = Date.now()
              trigger()
            }
            timeout = null
          }, interval - (now - lastUpdate))
        }
      }
    }
  }))
}
```

## Practical Example: Search Input

```vue
<script setup>
import { ref, watch } from 'vue'
import { useDebouncedRef } from '@/composables/useDebouncedRef'

const searchQuery = useDebouncedRef('', 400)
const results = ref([])
const loading = ref(false)

watch(searchQuery, async (query) => {
  if (!query) {
    results.value = []
    return
  }
  
  loading.value = true
  try {
    const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`)
    results.value = await response.json()
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <input v-model="searchQuery" placeholder="Search..." />
  <p v-if="loading">Searching...</p>
  <ul>
    <li v-for="result in results" :key="result.id">{{ result.name }}</li>
  </ul>
</template>
```

## Resources

- [customRef](https://vuejs.org/api/reactivity-advanced.html#customref) — Vue customRef API documentation

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*