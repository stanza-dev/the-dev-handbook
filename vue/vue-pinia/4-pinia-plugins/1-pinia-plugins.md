---
source_course: "vue-pinia"
source_lesson: "vue-pinia-pinia-plugins"
---

# Creating and Using Pinia Plugins

Pinia plugins let you extend stores with additional properties, methods, or behavior.

## Plugin Basics

A plugin is a function that receives context and can modify stores:

```typescript
import { PiniaPluginContext } from 'pinia'

export function myPlugin(context: PiniaPluginContext) {
  // context.app - Vue app instance
  // context.pinia - Pinia instance
  // context.store - Current store
  // context.options - Store options
  
  // Add a property to all stores
  context.store.hello = 'world'
  
  // Add a method to all stores
  context.store.greet = () => console.log('Hello from plugin!')
  
  // Return object to add to store
  return {
    secret: 'shhh'
  }
}

// Register plugin
const pinia = createPinia()
pinia.use(myPlugin)
```

## Adding Reactive State

```typescript
import { ref } from 'vue'
import { PiniaPluginContext } from 'pinia'

export function lastUpdatedPlugin({ store }: PiniaPluginContext) {
  // Add reactive property
  store.lastUpdated = ref<Date | null>(null)
  
  // Update on any state change
  store.$subscribe(() => {
    store.lastUpdated = new Date()
  })
}
```

## TypeScript: Augmenting Store Types

```typescript
// plugins/types.ts
import 'pinia'

declare module 'pinia' {
  export interface PiniaCustomProperties {
    lastUpdated: Ref<Date | null>
    greet: () => void
  }
  
  export interface PiniaCustomStateProperties<S> {
    // Add to state type
  }
  
  export interface DefineStoreOptionsBase<S, Store> {
    // Custom store options
    debounce?: Partial<Record<keyof StoreActions<Store>, number>>
  }
}
```

## Practical Plugin: Debounce Actions

```typescript
import { PiniaPluginContext } from 'pinia'
import debounce from 'lodash/debounce'

export function debouncePlugin({ options, store }: PiniaPluginContext) {
  if (options.debounce) {
    return Object.keys(options.debounce).reduce((acc, action) => {
      acc[action] = debounce(
        store[action],
        options.debounce![action]
      )
      return acc
    }, {} as Record<string, Function>)
  }
}

// Usage in store
export const useSearchStore = defineStore('search', {
  state: () => ({ query: '' }),
  actions: {
    search() { /* ... */ }
  },
  // Custom option
  debounce: {
    search: 300
  }
})
```

## Plugin: Reset All Stores

```typescript
import { PiniaPluginContext } from 'pinia'

export function resetPlugin({ pinia }: PiniaPluginContext) {
  // Add method to pinia instance
  if (!pinia._resetAll) {
    pinia._resetAll = () => {
      // Get all active stores
      pinia._s.forEach((store) => {
        store.$reset?.()
      })
    }
  }
  
  return {}
}

// Usage
const pinia = usePinia()
pinia._resetAll()  // Reset all stores
```

## Plugin: Loading State Manager

```typescript
import { ref, computed } from 'vue'
import { PiniaPluginContext } from 'pinia'

export function loadingPlugin({ store }: PiniaPluginContext) {
  const loadingActions = ref<Set<string>>(new Set())
  
  store.isLoading = computed(() => loadingActions.value.size > 0)
  store.loadingActions = loadingActions
  
  // Wrap actions
  const originalActions = { ...store.$actions }
  
  store.$onAction(({ name, after, onError }) => {
    // Track async actions
    const isAsync = originalActions[name]?.constructor.name === 'AsyncFunction'
    if (isAsync) {
      loadingActions.value.add(name)
      
      after(() => {
        loadingActions.value.delete(name)
      })
      
      onError(() => {
        loadingActions.value.delete(name)
      })
    }
  })
}
```

## Plugin: Undo/Redo

```typescript
import { PiniaPluginContext } from 'pinia'

export function historyPlugin({ store }: PiniaPluginContext) {
  const history: string[] = []
  const future: string[] = []
  let skipHistory = false
  
  // Save initial state
  history.push(JSON.stringify(store.$state))
  
  store.$subscribe((mutation, state) => {
    if (!skipHistory) {
      history.push(JSON.stringify(state))
      future.length = 0  // Clear redo stack
    }
  })
  
  store.canUndo = computed(() => history.length > 1)
  store.canRedo = computed(() => future.length > 0)
  
  store.undo = () => {
    if (history.length > 1) {
      future.push(history.pop()!)
      skipHistory = true
      store.$state = JSON.parse(history[history.length - 1])
      skipHistory = false
    }
  }
  
  store.redo = () => {
    if (future.length > 0) {
      const state = future.pop()!
      history.push(state)
      skipHistory = true
      store.$state = JSON.parse(state)
      skipHistory = false
    }
  }
}
```

## Resources

- [Plugins](https://pinia.vuejs.org/core-concepts/plugins.html) — Official documentation on Pinia plugins

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*