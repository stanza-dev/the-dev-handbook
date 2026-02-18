---
source_course: "vue-pinia"
source_lesson: "vue-pinia-store-patterns"
---

# Store Organization Patterns

Patterns for organizing Pinia stores in large applications.

## Store Categories

Organize stores by purpose:

```
stores/
├── modules/           # Feature stores
│   ├── useUserStore.ts
│   ├── useCartStore.ts
│   └── useProductStore.ts
├── ui/                # UI state
│   ├── useModalStore.ts
│   ├── useToastStore.ts
│   └── useSidebarStore.ts
└── app/               # App-level state
    ├── useAuthStore.ts
    └── useSettingsStore.ts
```

## Store Factories

Create stores dynamically:

```typescript
import { defineStore } from 'pinia'

export function createEntityStore<T extends { id: string | number }>(
  name: string
) {
  return defineStore(name, () => {
    const items = ref<Map<string | number, T>>(new Map())
    const loading = ref(false)
    const error = ref<string | null>(null)
    
    const all = computed(() => Array.from(items.value.values()))
    
    function getById(id: string | number) {
      return items.value.get(id)
    }
    
    function setItem(item: T) {
      items.value.set(item.id, item)
    }
    
    function setItems(newItems: T[]) {
      items.value.clear()
      newItems.forEach(item => items.value.set(item.id, item))
    }
    
    function removeItem(id: string | number) {
      items.value.delete(id)
    }
    
    return {
      items,
      loading,
      error,
      all,
      getById,
      setItem,
      setItems,
      removeItem
    }
  })
}

// Usage
export const useProductStore = createEntityStore<Product>('products')
export const useOrderStore = createEntityStore<Order>('orders')
```

## Store Composition

Compose stores from other stores:

```typescript
export const useCheckoutStore = defineStore('checkout', () => {
  const cartStore = useCartStore()
  const userStore = useUserStore()
  const orderStore = useOrderStore()
  
  const canCheckout = computed(() => 
    cartStore.itemCount > 0 && 
    userStore.isAuthenticated &&
    userStore.hasValidAddress
  )
  
  const orderSummary = computed(() => ({
    items: cartStore.items,
    subtotal: cartStore.subtotal,
    tax: cartStore.tax,
    shipping: calculateShipping(userStore.address),
    total: cartStore.total + calculateShipping(userStore.address)
  }))
  
  async function placeOrder() {
    if (!canCheckout.value) {
      throw new Error('Cannot checkout')
    }
    
    const order = await api.createOrder({
      items: cartStore.items,
      address: userStore.address,
      total: orderSummary.value.total
    })
    
    orderStore.setItem(order)
    cartStore.clear()
    
    return order
  }
  
  return {
    canCheckout,
    orderSummary,
    placeOrder
  }
})
```

## Normalized State

Store related data efficiently:

```typescript
interface NormalizedState {
  posts: Record<number, Post>
  users: Record<number, User>
  comments: Record<number, Comment>
  postIds: number[]
}

export const useBlogStore = defineStore('blog', () => {
  const posts = ref<Record<number, Post>>({})
  const users = ref<Record<number, User>>({})
  const comments = ref<Record<number, Comment>>({})
  const postIds = ref<number[]>([])
  
  // Denormalized view
  const postsWithAuthor = computed(() => 
    postIds.value.map(id => ({
      ...posts.value[id],
      author: users.value[posts.value[id].authorId],
      commentCount: Object.values(comments.value)
        .filter(c => c.postId === id).length
    }))
  )
  
  async function fetchPosts() {
    const response = await api.getPosts()
    
    // Normalize
    response.forEach(post => {
      posts.value[post.id] = post
      users.value[post.author.id] = post.author
      postIds.value.push(post.id)
    })
  }
  
  return { posts, users, comments, postIds, postsWithAuthor, fetchPosts }
})
```

## Resources

- [Pinia Best Practices](https://pinia.vuejs.org/cookbook/) — Pinia cookbook and best practices

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*