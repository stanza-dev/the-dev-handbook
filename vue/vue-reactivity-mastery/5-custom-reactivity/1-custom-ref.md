---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-custom-ref"
---

# Creating Custom Refs with customRef

`customRef()` lets you create refs with custom get/set behavior, enabling patterns like debouncing, validation, and transformation.

## Basic Structure

```typescript
import { customRef } from 'vue'

function myCustomRef<T>(initialValue: T) {
  return customRef<T>((track, trigger) => {
    let value = initialValue
    
    return {
      get() {
        track()  // Tell Vue to track this dependency
        return value
      },
      set(newValue) {
        value = newValue
        trigger()  // Tell Vue to re-run effects
      }
    }
  })
}
```

## Debounced Ref

The classic example—delay updates until typing stops:

```typescript
import { customRef } from 'vue'

function useDebouncedRef<T>(initialValue: T, delay = 300) {
  let timeout: number | null = null
  
  return customRef<T>((track, trigger) => {
    let value = initialValue
    
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        // Clear previous timeout
        if (timeout) {
          clearTimeout(timeout)
        }
        
        // Delay the update
        timeout = setTimeout(() => {
          value = newValue
          trigger()
        }, delay)
      }
    }
  })
}

// Usage
const searchQuery = useDebouncedRef('', 500)

// In template: <input v-model="searchQuery" />
// Effects only run 500ms after user stops typing
```

## Throttled Ref

Limit update frequency:

```typescript
import { customRef } from 'vue'

function useThrottledRef<T>(initialValue: T, interval = 100) {
  let lastUpdate = 0
  let pendingValue: T | null = null
  let timeout: number | null = null
  
  return customRef<T>((track, trigger) => {
    let value = initialValue
    
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        const now = Date.now()
        const timeSinceLastUpdate = now - lastUpdate
        
        if (timeSinceLastUpdate >= interval) {
          // Enough time passed, update immediately
          value = newValue
          lastUpdate = now
          trigger()
        } else {
          // Schedule update for later
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
            }, interval - timeSinceLastUpdate)
          }
        }
      }
    }
  })
}
```

## Validated Ref

Only accept valid values:

```typescript
import { customRef, ref } from 'vue'

type Validator<T> = (value: T) => boolean | string

function useValidatedRef<T>(initialValue: T, validator: Validator<T>) {
  const error = ref<string | null>(null)
  
  const valueRef = customRef<T>((track, trigger) => {
    let value = initialValue
    
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        const result = validator(newValue)
        
        if (result === true) {
          value = newValue
          error.value = null
          trigger()
        } else {
          error.value = typeof result === 'string' ? result : 'Invalid value'
        }
      }
    }
  })
  
  return { value: valueRef, error }
}

// Usage
const { value: age, error: ageError } = useValidatedRef(0, (v) => {
  if (v < 0) return 'Age cannot be negative'
  if (v > 150) return 'Age seems unrealistic'
  return true
})
```

## LocalStorage Synced Ref

```typescript
import { customRef, watch } from 'vue'

function useLocalStorageRef<T>(key: string, defaultValue: T) {
  // Load initial value
  const stored = localStorage.getItem(key)
  const initial: T = stored ? JSON.parse(stored) : defaultValue
  
  return customRef<T>((track, trigger) => {
    let value = initial
    
    // Listen for changes from other tabs
    window.addEventListener('storage', (e) => {
      if (e.key === key && e.newValue) {
        value = JSON.parse(e.newValue)
        trigger()
      }
    })
    
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        value = newValue
        localStorage.setItem(key, JSON.stringify(newValue))
        trigger()
      }
    }
  })
}

// Usage
const theme = useLocalStorageRef('theme', 'light')
theme.value = 'dark'  // Automatically saved to localStorage
```

## History Ref (Undo/Redo)

```typescript
import { customRef, ref, computed } from 'vue'

function useHistoryRef<T>(initialValue: T, maxHistory = 50) {
  const history = ref<T[]>([initialValue])
  const currentIndex = ref(0)
  
  const canUndo = computed(() => currentIndex.value > 0)
  const canRedo = computed(() => currentIndex.value < history.value.length - 1)
  
  const valueRef = customRef<T>((track, trigger) => {
    return {
      get() {
        track()
        return history.value[currentIndex.value]
      },
      set(newValue) {
        // Remove any "future" history
        history.value = history.value.slice(0, currentIndex.value + 1)
        
        // Add new value
        history.value.push(newValue)
        
        // Trim if too long
        if (history.value.length > maxHistory) {
          history.value.shift()
        } else {
          currentIndex.value++
        }
        
        trigger()
      }
    }
  })
  
  function undo() {
    if (canUndo.value) {
      currentIndex.value--
    }
  }
  
  function redo() {
    if (canRedo.value) {
      currentIndex.value++
    }
  }
  
  return {
    value: valueRef,
    history,
    canUndo,
    canRedo,
    undo,
    redo
  }
}
```

## Resources

- [customRef](https://vuejs.org/api/reactivity-advanced.html#customref) — Official API documentation for customRef

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*