---
source_course: "vue-pinia"
source_lesson: "vue-pinia-defining-stores"
---

# Defining Stores: Options vs Setup Syntax

Pinia offers two ways to define stores: Options syntax (similar to Vue Options API) and Setup syntax (similar to Composition API).

## Options Syntax

Familiar if you've used Vuex or Vue's Options API:

```typescript
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  // State is a function returning initial state
  state: () => ({
    count: 0,
    items: [] as string[],
    lastUpdated: null as Date | null
  }),
  
  // Getters are like computed properties
  getters: {
    // Arrow function with state parameter
    doubleCount: (state) => state.count * 2,
    
    // Regular function to access other getters via this
    quadrupleCount(): number {
      return this.doubleCount * 2
    },
    
    // Getter with parameters (returns a function)
    getItemByIndex: (state) => {
      return (index: number) => state.items[index]
    }
  },
  
  // Actions modify state
  actions: {
    increment() {
      this.count++
      this.lastUpdated = new Date()
    },
    
    addItem(item: string) {
      this.items.push(item)
    },
    
    // Async actions
    async fetchItems() {
      const response = await fetch('/api/items')
      this.items = await response.json()
    },
    
    // Reset to initial state
    reset() {
      this.$reset()
    }
  }
})
```

## Setup Syntax

Uses Composition API - more flexible and familiar to Vue 3 developers:

```typescript
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', () => {
  // State = ref()
  const count = ref(0)
  const items = ref<string[]>([])
  const lastUpdated = ref<Date | null>(null)
  
  // Getters = computed()
  const doubleCount = computed(() => count.value * 2)
  const quadrupleCount = computed(() => doubleCount.value * 2)
  const getItemByIndex = (index: number) => items.value[index]
  
  // Actions = functions
  function increment() {
    count.value++
    lastUpdated.value = new Date()
  }
  
  function addItem(item: string) {
    items.value.push(item)
  }
  
  async function fetchItems() {
    const response = await fetch('/api/items')
    items.value = await response.json()
  }
  
  function reset() {
    count.value = 0
    items.value = []
    lastUpdated.value = null
  }
  
  // Must return everything you want to expose
  return {
    // State
    count,
    items,
    lastUpdated,
    // Getters
    doubleCount,
    quadrupleCount,
    getItemByIndex,
    // Actions
    increment,
    addItem,
    fetchItems,
    reset
  }
})
```

## Comparison

| Feature | Options | Setup |
|---------|---------|-------|
| Learning curve | Easier | Requires Composition API knowledge |
| TypeScript | Good | Excellent |
| `this` access | ✅ Available | ❌ Not used |
| `$reset()` | ✅ Built-in | ❌ Must implement manually |
| Composable reuse | ❌ Limited | ✅ Can use other composables |
| Flexibility | Structured | More flexible |

## Setup Syntax Advantages

### Using Composables in Stores

```typescript
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'
import { useLocalStorage } from '@vueuse/core'

export const useSettingsStore = defineStore('settings', () => {
  // Use any composable!
  const theme = useLocalStorage('theme', 'light')
  const fontSize = useLocalStorage('fontSize', 14)
  
  const isDarkMode = computed(() => theme.value === 'dark')
  
  function toggleTheme() {
    theme.value = theme.value === 'light' ? 'dark' : 'light'
  }
  
  return {
    theme,
    fontSize,
    isDarkMode,
    toggleTheme
  }
})
```

### Using Watchers

```typescript
import { ref, watch } from 'vue'
import { defineStore } from 'pinia'

export const useFormStore = defineStore('form', () => {
  const email = ref('')
  const isValid = ref(false)
  
  // Watchers work in setup stores!
  watch(email, (newEmail) => {
    isValid.value = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(newEmail)
  })
  
  return { email, isValid }
})
```

## Recommendation

**Use Setup syntax** when:
- You need maximum flexibility
- You want to use composables
- Your team prefers Composition API
- You need watchers inside stores

**Use Options syntax** when:
- You prefer structured, object-based stores
- You want built-in `$reset()`
- You're migrating from Vuex
- Your team is more familiar with Options API

## Resources

- [Defining a Store](https://pinia.vuejs.org/core-concepts/) — Official guide on defining stores

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*