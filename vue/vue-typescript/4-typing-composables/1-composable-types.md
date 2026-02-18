---
source_course: "vue-typescript"
source_lesson: "vue-typescript-composable-types"
---

# Typing Composables

Well-typed composables provide excellent developer experience with autocomplete and type checking.

## Basic Composable Typing

```typescript
import { ref, computed, type Ref, type ComputedRef } from 'vue'

// Return type interface
interface UseCounterReturn {
  count: Ref<number>
  double: ComputedRef<number>
  increment: () => void
  decrement: () => void
  reset: () => void
}

export function useCounter(initialValue = 0): UseCounterReturn {
  const count = ref(initialValue)
  const double = computed(() => count.value * 2)
  
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  function reset() {
    count.value = initialValue
  }
  
  return {
    count,
    double,
    increment,
    decrement,
    reset
  }
}
```

## Accepting Ref or Value Arguments

```typescript
import { ref, unref, computed, type Ref, type MaybeRef } from 'vue'

export function useDoubled(value: MaybeRef<number>) {
  // MaybeRef<T> = T | Ref<T>
  // unref() handles both cases
  return computed(() => unref(value) * 2)
}

// Usage - both work
useDoubled(5)          // Plain number
useDoubled(ref(5))     // Ref
useDoubled(count)      // Existing ref
```

## MaybeRefOrGetter Pattern

```typescript
import { toValue, type MaybeRefOrGetter } from 'vue'

// MaybeRefOrGetter<T> = T | Ref<T> | (() => T)
export function useTitle(title: MaybeRefOrGetter<string>) {
  watchEffect(() => {
    // toValue handles all three cases
    document.title = toValue(title)
  })
}

// All of these work:
useTitle('Static Title')
useTitle(ref('Reactive Title'))
useTitle(() => `Page ${count.value}`)
```

## Async Composable

```typescript
import { ref, shallowRef, type Ref, type ShallowRef } from 'vue'

interface UseFetchOptions<T> {
  immediate?: boolean
  initialData?: T
}

interface UseFetchReturn<T> {
  data: ShallowRef<T | null>
  error: ShallowRef<Error | null>
  loading: Ref<boolean>
  execute: () => Promise<void>
}

export function useFetch<T>(
  url: MaybeRefOrGetter<string>,
  options: UseFetchOptions<T> = {}
): UseFetchReturn<T> {
  const { immediate = true, initialData = null } = options
  
  const data = shallowRef<T | null>(initialData)
  const error = shallowRef<Error | null>(null)
  const loading = ref(false)
  
  async function execute(): Promise<void> {
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(toValue(url))
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
  
  if (immediate) {
    execute()
  }
  
  return { data, error, loading, execute }
}
```

## Generic Composable

```typescript
import { ref, type Ref } from 'vue'

interface UseLocalStorageOptions<T> {
  serializer?: {
    read: (raw: string) => T
    write: (value: T) => string
  }
}

export function useLocalStorage<T>(
  key: string,
  defaultValue: T,
  options: UseLocalStorageOptions<T> = {}
): Ref<T> {
  const {
    serializer = {
      read: JSON.parse,
      write: JSON.stringify
    }
  } = options
  
  const stored = localStorage.getItem(key)
  const data = ref<T>(
    stored !== null ? serializer.read(stored) : defaultValue
  ) as Ref<T>
  
  watch(
    data,
    (value) => {
      localStorage.setItem(key, serializer.write(value))
    },
    { deep: true }
  )
  
  return data
}

// Usage with type inference
const theme = useLocalStorage('theme', 'light')  // Ref<string>
const count = useLocalStorage('count', 0)        // Ref<number>
const user = useLocalStorage<User>('user', { name: '', email: '' })
```

## Composable with Cleanup

```typescript
import { ref, onMounted, onUnmounted, type Ref } from 'vue'

interface MousePosition {
  x: Ref<number>
  y: Ref<number>
}

export function useMouse(): MousePosition {
  const x = ref(0)
  const y = ref(0)
  
  function update(event: MouseEvent): void {
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

## Composable Returning Tuple

```typescript
import { ref, type Ref } from 'vue'

type UseToggleReturn = [Ref<boolean>, () => void, (value: boolean) => void]

export function useToggle(initialValue = false): UseToggleReturn {
  const value = ref(initialValue)
  
  function toggle(): void {
    value.value = !value.value
  }
  
  function set(newValue: boolean): void {
    value.value = newValue
  }
  
  return [value, toggle, set]
}

// Usage
const [isOpen, toggleOpen, setOpen] = useToggle(false)
```

## Resources

- [Composables](https://vuejs.org/guide/reusability/composables.html) — Official guide on composables

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*