---
source_course: "vue-pinia"
source_lesson: "vue-pinia-accessing-state"
---

# Accessing and Modifying State

Understanding how to properly access and modify Pinia state is crucial for building reactive applications.

## Accessing State

### Direct Access

```vue
<script setup>
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

// Access state directly
console.log(userStore.name)     // 'Alice'
console.log(userStore.isAdmin)  // true
</script>

<template>
  <!-- State is reactive in templates -->
  <p>Welcome, {{ userStore.name }}</p>
  <span v-if="userStore.isAdmin">Admin</span>
</template>
```

### Destructuring with storeToRefs

Direct destructuring loses reactivity! Use `storeToRefs`:

```vue
<script setup>
import { useUserStore } from '@/stores/user'
import { storeToRefs } from 'pinia'

const userStore = useUserStore()

// ❌ Loses reactivity!
const { name, email } = userStore

// ✅ Keeps reactivity
const { name, email } = storeToRefs(userStore)

// Actions can be destructured directly (they're functions)
const { updateProfile, logout } = userStore
</script>
```

## Modifying State

### Direct Mutation

```typescript
const store = useCounterStore()

// Direct mutation is allowed!
store.count++
store.name = 'New Name'
store.items.push('new item')
```

### Using $patch()

Better for multiple mutations - groups them as a single entry in devtools:

```typescript
const store = useUserStore()

// Object syntax
store.$patch({
  name: 'Alice',
  email: 'alice@example.com',
  age: 30
})

// Function syntax - for complex mutations
store.$patch((state) => {
  state.items.push('new item')
  state.total = state.items.length
  state.hasItems = state.items.length > 0
})
```

### Using Actions

Recommended for complex logic or async operations:

```typescript
// In store definition
actions: {
  updateUser(userData: Partial<User>) {
    this.name = userData.name ?? this.name
    this.email = userData.email ?? this.email
    this.lastUpdated = new Date()
  }
}

// Usage
store.updateUser({ name: 'Bob', email: 'bob@example.com' })
```

## Resetting State

### Options Syntax: $reset()

```typescript
const store = useCounterStore()

store.count = 100
store.$reset()  // count is back to 0
```

### Setup Syntax: Manual Reset

```typescript
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const name = ref('Counter')
  
  // Implement reset manually
  function $reset() {
    count.value = 0
    name.value = 'Counter'
  }
  
  return { count, name, $reset }
})
```

## Replacing Entire State

Use `$state` for complete replacement:

```typescript
const store = useUserStore()

// Replace entire state
store.$state = {
  name: 'New User',
  email: 'new@example.com',
  settings: { theme: 'dark' }
}
```

## Subscribing to State Changes

React to state changes outside of Vue components:

```typescript
const store = useUserStore()

// Subscribe to all state changes
const unsubscribe = store.$subscribe((mutation, state) => {
  console.log('Mutation type:', mutation.type)
  console.log('Store ID:', mutation.storeId)
  console.log('New state:', state)
  
  // Persist to localStorage
  localStorage.setItem('user', JSON.stringify(state))
})

// Options
store.$subscribe(
  (mutation, state) => { /* ... */ },
  { detached: true }  // Survives component unmount
)

// Unsubscribe
unsubscribe()
```

## State TypeScript Tips

```typescript
import { defineStore } from 'pinia'

// Define state type separately for reuse
type UserState = {
  id: number | null
  name: string
  email: string
  roles: string[]
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    id: null,
    name: '',
    email: '',
    roles: []
  }),
  
  getters: {
    // Type is inferred from state
    isLoggedIn: (state) => state.id !== null,
    
    // Explicit return type when needed
    primaryRole(): string | null {
      return this.roles[0] ?? null
    }
  }
})
```

## Resources

- [State](https://pinia.vuejs.org/core-concepts/state.html) — Official documentation on Pinia state

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*