---
source_course: "vue-pinia"
source_lesson: "vue-pinia-state-persistence"
---

# Persisting State

Persist Pinia store state to localStorage, sessionStorage, or other storage mechanisms.

## Manual Persistence

### Simple localStorage Sync

```typescript
import { defineStore } from 'pinia'

export const useSettingsStore = defineStore('settings', {
  state: () => {
    // Load from localStorage on initialization
    const stored = localStorage.getItem('settings')
    return stored ? JSON.parse(stored) : {
      theme: 'light',
      fontSize: 14,
      language: 'en'
    }
  },
  
  actions: {
    setTheme(theme: string) {
      this.theme = theme
      this.save()
    },
    
    save() {
      localStorage.setItem('settings', JSON.stringify(this.$state))
    }
  }
})
```

### Using $subscribe

```typescript
const store = useSettingsStore()

// Auto-save on any change
store.$subscribe((mutation, state) => {
  localStorage.setItem('settings', JSON.stringify(state))
})
```

## Persistence Plugin

Create a reusable persistence plugin:

```typescript
import { PiniaPluginContext, StateTree } from 'pinia'

type PersistOptions = {
  key?: string
  storage?: Storage
  paths?: string[]
}

export function persistPlugin(context: PiniaPluginContext) {
  const { store, options } = context
  
  // Check if store has persist options
  const persist = (options as any).persist as PersistOptions | boolean
  if (!persist) return
  
  const opts: PersistOptions = typeof persist === 'boolean' ? {} : persist
  const key = opts.key ?? store.$id
  const storage = opts.storage ?? localStorage
  const paths = opts.paths
  
  // Restore state
  const stored = storage.getItem(key)
  if (stored) {
    try {
      const parsed = JSON.parse(stored)
      store.$patch(parsed)
    } catch (e) {
      console.error('Failed to restore state:', e)
    }
  }
  
  // Save on changes
  store.$subscribe((_, state) => {
    const toStore = paths 
      ? paths.reduce((acc, path) => {
          const value = path.split('.').reduce((obj, key) => obj?.[key], state as any)
          if (value !== undefined) {
            acc[path] = value
          }
          return acc
        }, {} as Record<string, any>)
      : state
    
    storage.setItem(key, JSON.stringify(toStore))
  })
}

// Usage
export const useUserStore = defineStore('user', {
  state: () => ({
    name: '',
    email: '',
    token: '',  // Sensitive - maybe don't persist
    preferences: {}
  }),
  
  persist: {
    key: 'user-data',
    storage: localStorage,
    paths: ['name', 'email', 'preferences']  // Only persist these
  }
})
```

## pinia-plugin-persistedstate

The recommended solution for persistence:

```bash
npm install pinia-plugin-persistedstate
```

```typescript
// main.ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)
```

```typescript
// store
export const useUserStore = defineStore('user', {
  state: () => ({
    name: '',
    email: '',
    token: ''
  }),
  
  persist: true  // Simple: persist everything
})

// Advanced options
export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [],
    lastUpdated: null
  }),
  
  persist: {
    key: 'shopping-cart',
    storage: sessionStorage,  // Use sessionStorage
    paths: ['items'],  // Only persist items
    beforeRestore: (ctx) => {
      console.log('About to restore', ctx.store.$id)
    },
    afterRestore: (ctx) => {
      console.log('Restored', ctx.store.$id)
    }
  }
})
```

## Setup Syntax with Persistence

```typescript
import { ref } from 'vue'
import { defineStore } from 'pinia'

export const useAuthStore = defineStore('auth', () => {
  const token = ref('')
  const user = ref(null)
  
  function setToken(newToken: string) {
    token.value = newToken
  }
  
  function logout() {
    token.value = ''
    user.value = null
  }
  
  return { token, user, setToken, logout }
}, {
  persist: {
    paths: ['token']
  }
})
```

## Custom Serialization

```typescript
export const useStore = defineStore('store', {
  state: () => ({
    date: new Date(),
    map: new Map()
  }),
  
  persist: {
    serializer: {
      serialize: (state) => JSON.stringify({
        ...state,
        date: state.date.toISOString(),
        map: Array.from(state.map.entries())
      }),
      deserialize: (str) => {
        const parsed = JSON.parse(str)
        return {
          ...parsed,
          date: new Date(parsed.date),
          map: new Map(parsed.map)
        }
      }
    }
  }
})
```

## Resources

- [pinia-plugin-persistedstate](https://prazdevs.github.io/pinia-plugin-persistedstate/) — Popular persistence plugin for Pinia

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*