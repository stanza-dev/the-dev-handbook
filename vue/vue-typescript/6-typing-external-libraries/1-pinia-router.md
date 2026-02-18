---
source_course: "vue-typescript"
source_lesson: "vue-typescript-typing-pinia-router"
---

# Typing Pinia and Vue Router

Both Pinia and Vue Router have excellent TypeScript support. Learn to leverage their type systems.

## Typing Pinia Stores

### Setup Store (Fully Typed)

```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
  email: string
  role: 'user' | 'admin'
}

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  const isAuthenticated = computed(() => user.value !== null)
  const isAdmin = computed(() => user.value?.role === 'admin')
  
  async function login(email: string, password: string): Promise<boolean> {
    loading.value = true
    error.value = null
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password })
      })
      if (!response.ok) throw new Error('Login failed')
      user.value = await response.json()
      return true
    } catch (e) {
      error.value = (e as Error).message
      return false
    } finally {
      loading.value = false
    }
  }
  
  function logout() {
    user.value = null
  }
  
  return {
    user,
    loading,
    error,
    isAuthenticated,
    isAdmin,
    login,
    logout
  }
})

// Type is inferred automatically
type UserStore = ReturnType<typeof useUserStore>
```

### Options Store with Types

```typescript
import { defineStore } from 'pinia'

interface CartItem {
  productId: number
  name: string
  price: number
  quantity: number
}

interface CartState {
  items: CartItem[]
  couponCode: string | null
}

export const useCartStore = defineStore('cart', {
  state: (): CartState => ({
    items: [],
    couponCode: null
  }),
  
  getters: {
    itemCount: (state): number => state.items.length,
    
    total(): number {
      return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
    },
    
    getItemByProductId: (state) => {
      return (productId: number): CartItem | undefined =>
        state.items.find(item => item.productId === productId)
    }
  },
  
  actions: {
    addItem(item: Omit<CartItem, 'quantity'>): void {
      const existing = this.getItemByProductId(item.productId)
      if (existing) {
        existing.quantity++
      } else {
        this.items.push({ ...item, quantity: 1 })
      }
    }
  }
})
```

## Typing Vue Router

### Typed Route Records

```typescript
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'home',
    component: () => import('./views/Home.vue')
  },
  {
    path: '/users/:id',
    name: 'user',
    component: () => import('./views/User.vue'),
    props: true
  }
]
```

### Extending Route Meta

```typescript
// router/types.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    roles?: ('user' | 'admin')[]
    title?: string
    layout?: 'default' | 'admin' | 'auth'
  }
}
```

### Typed Route Params

```vue
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()

// Type assertion for known params
const userId = route.params.id as string

// Or with type guard
function assertStringParam(param: string | string[]): string {
  return Array.isArray(param) ? param[0] : param
}

const safeUserId = assertStringParam(route.params.id)
</script>
```

## Resources

- [Pinia TypeScript](https://pinia.vuejs.org/core-concepts/state.html#typescript) — Pinia TypeScript support
- [Vue Router TypeScript](https://router.vuejs.org/guide/advanced/typed-routes.html) — Vue Router TypeScript support

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*