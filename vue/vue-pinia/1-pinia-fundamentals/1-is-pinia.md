---
source_course: "vue-pinia"
source_lesson: "vue-pinia-what-is-pinia"
---

# What is Pinia?

Pinia is the official state management library for Vue 3. It replaces Vuex as the recommended solution and provides a simpler, more intuitive API with full TypeScript support.

## Why Pinia?

### Pinia vs Vuex Comparison

| Feature | Pinia | Vuex 4 |
|---------|-------|--------|
| Mutations | ❌ No mutations needed | ✅ Required for state changes |
| Modules | ❌ Flat stores (no nesting) | ✅ Nested modules |
| TypeScript | ✅ First-class support | 🟡 Requires extra setup |
| Devtools | ✅ Full support | ✅ Full support |
| Bundle size | ~1.5kb | ~10kb |
| API style | Options & Setup | Options only |

### Key Benefits

1. **No more mutations**: Actions directly modify state
2. **No modules**: Flat store structure, import what you need
3. **TypeScript**: Full type inference out of the box
4. **Devtools support**: Time travel, editing, timeline
5. **Hot Module Replacement**: Edit stores without page reload
6. **SSR Ready**: Works with server-side rendering

## When Do You Need Pinia?

### ✅ Use Pinia When:

- State needs to be shared across multiple components
- State must survive component unmounting
- Application has complex data flows
- You need time-travel debugging
- Working with authentication/user state
- Shopping cart, notifications, or similar global state

### ❌ Don't Need Pinia When:

- State is local to a component (use `ref`/`reactive`)
- Parent-child communication only (use props/emits)
- Simple shared state (consider `provide`/`inject`)
- Component tree state (consider composables)

## Installation

```bash
npm install pinia
# or
pnpm add pinia
# or
yarn add pinia
```

## Setup

```typescript
// main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.mount('#app')
```

## Your First Store

```typescript
// stores/counter.ts
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  // State - the data
  state: () => ({
    count: 0,
    name: 'My Counter'
  }),
  
  // Getters - computed properties
  getters: {
    doubleCount: (state) => state.count * 2
  },
  
  // Actions - methods that modify state
  actions: {
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    }
  }
})
```

## Using the Store

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()
</script>

<template>
  <div>
    <p>{{ counter.name }}: {{ counter.count }}</p>
    <p>Double: {{ counter.doubleCount }}</p>
    <button @click="counter.increment">+</button>
    <button @click="counter.decrement">-</button>
  </div>
</template>
```

## Store Naming Convention

Always name stores with `use` prefix and `Store` suffix:

```typescript
// ✅ Good
useUserStore
useCartStore
useAuthStore
useNotificationStore

// ❌ Bad
user
cartStore
auth
```

## Store Organization

```
src/
├── stores/
│   ├── index.ts       # Optional: re-export all stores
│   ├── user.ts        # User/auth store
│   ├── cart.ts        # Shopping cart store
│   ├── products.ts    # Products store
│   └── ui.ts          # UI state (sidebar, modals, etc.)
```

## Resources

- [Pinia Documentation](https://pinia.vuejs.org/) — Official Pinia documentation

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*