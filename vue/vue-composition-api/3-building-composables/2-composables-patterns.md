---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-composables-patterns"
---

# Advanced Composable Patterns

Let's explore powerful patterns for building production-ready composables.

## Async Composables with useFetch

```typescript
import { ref, unref, watchEffect, type Ref } from 'vue'

type MaybeRef<T> = T | Ref<T>

export function useFetch<T>(url: MaybeRef<string>) {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)
  
  async function execute() {
    data.value = null
    error.value = null
    loading.value = true
    
    try {
      const response = await fetch(unref(url))
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`)
      }
      data.value = await response.json()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }
  
  // Auto-fetch when URL changes
  watchEffect(() => {
    execute()
  })
  
  return {
    data,
    error,
    loading,
    refetch: execute
  }
}
```

```vue
<script setup>
import { ref, computed } from 'vue'
import { useFetch } from './useFetch'

const userId = ref(1)
const url = computed(() => `/api/users/${userId.value}`)

// Auto-refetches when userId changes
const { data: user, loading, error, refetch } = useFetch(url)
</script>
```

## Accepting Ref or Value Arguments

```typescript
import { ref, unref, computed, type MaybeRef } from 'vue'

export function useTitle(title: MaybeRef<string>) {
  // Works with both ref and plain string
  const titleRef = computed(() => unref(title))
  
  watchEffect(() => {
    document.title = titleRef.value
  })
  
  return titleRef
}

// Usage - both work:
useTitle('Static Title')
useTitle(ref('Dynamic Title'))
useTitle(computed(() => `Page ${count.value}`))
```

## Returning Both Single Ref and Object

```typescript
export function useCounter(initial = 0) {
  const count = ref(initial)
  
  function increment() { count.value++ }
  function decrement() { count.value-- }
  function reset() { count.value = initial }
  
  // Return object for full API
  return {
    count,
    increment,
    decrement,
    reset
  }
}

// If you only need the count:
const { count } = useCounter()

// Full API:
const counter = useCounter()
counter.increment()
```

## Configurable Composables

```typescript
type UseFetchOptions<T> = {
  immediate?: boolean
  initialData?: T
  refetchOnWindowFocus?: boolean
  transform?: (data: unknown) => T
  onError?: (error: Error) => void
}

export function useFetch<T>(
  url: MaybeRef<string>,
  options: UseFetchOptions<T> = {}
) {
  const {
    immediate = true,
    initialData = null,
    refetchOnWindowFocus = false,
    transform = (d) => d as T,
    onError
  } = options
  
  const data = ref<T | null>(initialData)
  const error = ref<Error | null>(null)
  const loading = ref(false)
  
  async function execute() {
    loading.value = true
    try {
      const response = await fetch(unref(url))
      data.value = transform(await response.json())
    } catch (e) {
      error.value = e as Error
      onError?.(error.value)
    } finally {
      loading.value = false
    }
  }
  
  if (immediate) {
    execute()
  }
  
  if (refetchOnWindowFocus) {
    onMounted(() => {
      window.addEventListener('focus', execute)
    })
    onUnmounted(() => {
      window.removeEventListener('focus', execute)
    })
  }
  
  return { data, error, loading, execute }
}
```

## State Machine Composable

```typescript
import { ref, computed } from 'vue'

type State = 'idle' | 'loading' | 'success' | 'error'

export function useAsyncState<T>() {
  const state = ref<State>('idle')
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  
  const isIdle = computed(() => state.value === 'idle')
  const isLoading = computed(() => state.value === 'loading')
  const isSuccess = computed(() => state.value === 'success')
  const isError = computed(() => state.value === 'error')
  
  async function execute(promise: Promise<T>) {
    state.value = 'loading'
    error.value = null
    
    try {
      data.value = await promise
      state.value = 'success'
      return data.value
    } catch (e) {
      error.value = e as Error
      state.value = 'error'
      throw e
    }
  }
  
  function reset() {
    state.value = 'idle'
    data.value = null
    error.value = null
  }
  
  return {
    state,
    data,
    error,
    isIdle,
    isLoading,
    isSuccess,
    isError,
    execute,
    reset
  }
}
```

## Shared State Composable

```typescript
import { ref } from 'vue'

// State is defined outside the function - shared!
const globalCount = ref(0)

export function useSharedCounter() {
  function increment() {
    globalCount.value++
  }
  
  return {
    count: globalCount,
    increment
  }
}

// Every component using useSharedCounter shares the same count
```

### With Lazy Initialization

```typescript
import { ref, type Ref } from 'vue'

let sharedState: Ref<User | null> | null = null

export function useCurrentUser() {
  // Initialize only once
  if (!sharedState) {
    sharedState = ref(null)
    // Fetch user on first use
    fetchCurrentUser().then(user => {
      sharedState!.value = user
    })
  }
  
  return sharedState
}
```

## Resources

- [VueUse - Collection of Composables](https://vueuse.org/) — Comprehensive collection of Vue composables

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*