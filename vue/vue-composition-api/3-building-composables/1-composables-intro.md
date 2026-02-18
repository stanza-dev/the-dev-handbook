---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-composables-intro"
---

# What are Composables?

Composables are functions that use Vue's Composition API to encapsulate and reuse stateful logic. They're the Vue 3 equivalent of mixins, but better.

## Composable vs Mixin

### Mixins (Vue 2 style - problematic)

```javascript
// mixin.js
export const counterMixin = {
  data() {
    return { count: 0 }  // Where does 'count' come from in component?
  },
  methods: {
    increment() { this.count++ }  // Name conflicts?
  }
}

// component
export default {
  mixins: [counterMixin, anotherMixin],  // What if both have 'count'?
  // Can't tell which mixin provides what
}
```

### Composables (Vue 3 - clear and explicit)

```typescript
// useCounter.ts
import { ref } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  return { count, increment, decrement }
}

// component
<script setup>
import { useCounter } from './useCounter'

// Explicit destructuring - clear what's being used
const { count, increment } = useCounter(10)
const { count: otherCount, increment: otherIncrement } = useCounter(0)
// No conflicts, easy renaming!
</script>
```

## Anatomy of a Composable

```typescript
import { ref, computed, onMounted, onUnmounted } from 'vue'

export function useFeature(options = {}) {
  // 1. Reactive State
  const state = ref(initialValue)
  const loading = ref(false)
  const error = ref<Error | null>(null)
  
  // 2. Computed Properties
  const derived = computed(() => state.value * 2)
  
  // 3. Methods
  function doSomething() {
    state.value++
  }
  
  async function fetchData() {
    loading.value = true
    try {
      state.value = await api.getData()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }
  
  // 4. Lifecycle Hooks
  onMounted(() => {
    // Setup
  })
  
  onUnmounted(() => {
    // Cleanup
  })
  
  // 5. Return Values
  return {
    // State (refs)
    state,
    loading,
    error,
    derived,
    // Methods
    doSomething,
    fetchData
  }
}
```

## Naming Convention

Composables should start with `use`:

```typescript
// ✅ Good names
useCounter()
useMouse()
useLocalStorage()
useFetch()
useAuth()

// ❌ Bad names
counter()
getMouse()
localStorageHelper()
```

## Example: useMouse

```typescript
// useMouse.ts
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)
  
  function update(event: MouseEvent) {
    x.value = event.clientX
    y.value = event.clientY
  }
  
  onMounted(() => {
    window.addEventListener('mousemove', update)
  })
  
  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })
  
  return { x, y }
}
```

```vue
<!-- Usage -->
<script setup>
import { useMouse } from './useMouse'

const { x, y } = useMouse()
</script>

<template>
  <p>Mouse: {{ x }}, {{ y }}</p>
</template>
```

## Example: useLocalStorage

```typescript
// useLocalStorage.ts
import { ref, watch } from 'vue'

export function useLocalStorage<T>(key: string, defaultValue: T) {
  // Read from localStorage or use default
  const stored = localStorage.getItem(key)
  const value = ref<T>(
    stored !== null ? JSON.parse(stored) : defaultValue
  )
  
  // Sync to localStorage on changes
  watch(
    value,
    (newValue) => {
      if (newValue === null) {
        localStorage.removeItem(key)
      } else {
        localStorage.setItem(key, JSON.stringify(newValue))
      }
    },
    { deep: true }
  )
  
  return value
}
```

```vue
<!-- Usage -->
<script setup>
import { useLocalStorage } from './useLocalStorage'

const theme = useLocalStorage('theme', 'light')
const savedItems = useLocalStorage('cart', [])

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}
</script>
```

## Composables Can Use Other Composables

```typescript
// useMouseInElement.ts
import { ref, computed } from 'vue'
import { useMouse } from './useMouse'
import { useElementBounds } from './useElementBounds'

export function useMouseInElement(target: Ref<HTMLElement | null>) {
  const { x: mouseX, y: mouseY } = useMouse()
  const { left, top, width, height } = useElementBounds(target)
  
  const x = computed(() => mouseX.value - left.value)
  const y = computed(() => mouseY.value - top.value)
  
  const isInside = computed(() =>
    x.value >= 0 && 
    x.value <= width.value && 
    y.value >= 0 && 
    y.value <= height.value
  )
  
  return { x, y, isInside }
}
```

## Resources

- [Composables](https://vuejs.org/guide/reusability/composables.html) — Official guide to composables

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*