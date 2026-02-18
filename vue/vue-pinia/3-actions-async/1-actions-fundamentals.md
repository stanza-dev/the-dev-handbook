---
source_course: "vue-pinia"
source_lesson: "vue-pinia-actions-fundamentals"
---

# Actions Fundamentals

Actions in Pinia are methods that modify state. Unlike Vuex, there's no distinction between mutations and actions—actions can directly modify state.

## Basic Actions

### Options Syntax

```typescript
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0,
    history: [] as number[]
  }),
  
  actions: {
    // Simple action
    increment() {
      this.count++
    },
    
    // Action with parameters
    incrementBy(amount: number) {
      this.count += amount
    },
    
    // Action with validation
    setCount(value: number) {
      if (value < 0) {
        console.warn('Count cannot be negative')
        return
      }
      this.history.push(this.count)
      this.count = value
    },
    
    // Action using getters
    doubleAndIncrement() {
      this.count = this.doubleCount + 1  // this.doubleCount is a getter
    },
    
    // Action calling other actions
    reset() {
      this.setCount(0)
      this.history = []
    }
  }
})
```

### Setup Syntax

```typescript
import { ref } from 'vue'
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const history = ref<number[]>([])
  
  function increment() {
    count.value++
  }
  
  function incrementBy(amount: number) {
    count.value += amount
  }
  
  function setCount(value: number) {
    if (value < 0) {
      console.warn('Count cannot be negative')
      return
    }
    history.value.push(count.value)
    count.value = value
  }
  
  function reset() {
    setCount(0)
    history.value = []
  }
  
  return { count, history, increment, incrementBy, setCount, reset }
})
```

## Returning Values from Actions

```typescript
actions: {
  // Return success/failure
  addItem(item: Item): boolean {
    if (this.items.length >= this.maxItems) {
      return false
    }
    this.items.push(item)
    return true
  },
  
  // Return the created item
  createItem(data: ItemData): Item {
    const item = {
      id: Date.now(),
      ...data,
      createdAt: new Date()
    }
    this.items.push(item)
    return item
  }
}

// Usage
const success = store.addItem(newItem)
if (!success) {
  alert('Cart is full!')
}

const created = store.createItem({ name: 'New Item' })
console.log('Created:', created.id)
```

## Actions Accessing Other Stores

```typescript
import { defineStore } from 'pinia'
import { useUserStore } from './user'
import { useNotificationStore } from './notification'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [] as CartItem[]
  }),
  
  actions: {
    async checkout() {
      const userStore = useUserStore()
      const notificationStore = useNotificationStore()
      
      if (!userStore.isLoggedIn) {
        notificationStore.show('Please log in to checkout', 'error')
        return
      }
      
      try {
        await api.checkout(this.items, userStore.id)
        this.items = []
        notificationStore.show('Order placed!', 'success')
      } catch (error) {
        notificationStore.show('Checkout failed', 'error')
      }
    }
  }
})
```

## Subscribing to Actions

Listen to actions being called:

```typescript
const store = useCartStore()

const unsubscribe = store.$onAction(({ 
  name,       // Action name
  store,      // Store instance
  args,       // Arguments passed to action
  after,      // Hook after action returns
  onError     // Hook if action throws
}) => {
  const startTime = Date.now()
  console.log(`Action "${name}" started with args:`, args)
  
  after((result) => {
    console.log(
      `Action "${name}" finished in ${Date.now() - startTime}ms`,
      'Result:', result
    )
  })
  
  onError((error) => {
    console.error(`Action "${name}" failed:`, error)
  })
})

// Unsubscribe when needed
unsubscribe()

// Keep subscription after component unmounts
store.$onAction(callback, true)  // detached: true
```

## Error Handling in Actions

```typescript
import { defineStore } from 'pinia'

export const useDataStore = defineStore('data', {
  state: () => ({
    data: null as Data | null,
    error: null as Error | null,
    loading: false
  }),
  
  actions: {
    async fetchData() {
      this.loading = true
      this.error = null
      
      try {
        const response = await fetch('/api/data')
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`)
        }
        this.data = await response.json()
      } catch (e) {
        this.error = e as Error
        throw e  // Re-throw for caller to handle
      } finally {
        this.loading = false
      }
    }
  }
})
```

## Resources

- [Actions](https://pinia.vuejs.org/core-concepts/actions.html) — Official documentation on Pinia actions

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*