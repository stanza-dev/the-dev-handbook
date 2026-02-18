---
source_course: "vue-pinia"
source_lesson: "vue-pinia-testing-stores"
---

# Testing Pinia Stores

Pinia stores are easy to test because they're just functions. You can test them in isolation or with components.

## Setup for Testing

```typescript
// vitest.config.ts or jest setup
import { beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'

beforeEach(() => {
  // Create a fresh pinia for each test
  setActivePinia(createPinia())
})
```

## Testing State

```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useCounterStore } from '@/stores/counter'

describe('Counter Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('has initial state', () => {
    const store = useCounterStore()
    
    expect(store.count).toBe(0)
    expect(store.name).toBe('Counter')
  })
  
  it('can modify state directly', () => {
    const store = useCounterStore()
    
    store.count = 10
    
    expect(store.count).toBe(10)
  })
})
```

## Testing Actions

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useCartStore } from '@/stores/cart'

describe('Cart Store Actions', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('adds item to cart', () => {
    const store = useCartStore()
    
    store.addItem({ id: 1, name: 'Product', price: 10 })
    
    expect(store.items).toHaveLength(1)
    expect(store.items[0].name).toBe('Product')
  })
  
  it('removes item from cart', () => {
    const store = useCartStore()
    store.items = [{ id: 1, name: 'Product', price: 10 }]
    
    store.removeItem(1)
    
    expect(store.items).toHaveLength(0)
  })
  
  it('clears cart', () => {
    const store = useCartStore()
    store.items = [
      { id: 1, name: 'A', price: 10 },
      { id: 2, name: 'B', price: 20 }
    ]
    
    store.clearCart()
    
    expect(store.items).toHaveLength(0)
  })
})
```

## Testing Getters

```typescript
describe('Cart Store Getters', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('calculates total correctly', () => {
    const store = useCartStore()
    store.items = [
      { id: 1, name: 'A', price: 10, quantity: 2 },
      { id: 2, name: 'B', price: 5, quantity: 3 }
    ]
    
    expect(store.total).toBe(35)  // 10*2 + 5*3
  })
  
  it('returns empty total for empty cart', () => {
    const store = useCartStore()
    
    expect(store.total).toBe(0)
  })
  
  it('counts items correctly', () => {
    const store = useCartStore()
    store.items = [
      { id: 1, quantity: 2 },
      { id: 2, quantity: 3 }
    ]
    
    expect(store.itemCount).toBe(5)
  })
})
```

## Testing Async Actions

```typescript
import { vi } from 'vitest'

describe('Async Actions', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
    vi.clearAllMocks()
  })
  
  it('fetches data successfully', async () => {
    const mockData = [{ id: 1, name: 'Product' }]
    
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(mockData)
    })
    
    const store = useProductStore()
    await store.fetchProducts()
    
    expect(store.products).toEqual(mockData)
    expect(store.loading).toBe(false)
    expect(store.error).toBeNull()
  })
  
  it('handles fetch error', async () => {
    global.fetch = vi.fn().mockRejectedValue(new Error('Network error'))
    
    const store = useProductStore()
    
    await expect(store.fetchProducts()).rejects.toThrow()
    expect(store.error).toBe('Network error')
    expect(store.loading).toBe(false)
  })
  
  it('sets loading state during fetch', async () => {
    let resolvePromise: Function
    const promise = new Promise(resolve => { resolvePromise = resolve })
    
    global.fetch = vi.fn().mockReturnValue({
      ok: true,
      json: () => promise
    })
    
    const store = useProductStore()
    const fetchPromise = store.fetchProducts()
    
    expect(store.loading).toBe(true)
    
    resolvePromise!([])
    await fetchPromise
    
    expect(store.loading).toBe(false)
  })
})
```

## Testing with Components

```typescript
import { mount } from '@vue/test-utils'
import { createTestingPinia } from '@pinia/testing'
import Counter from '@/components/Counter.vue'

describe('Counter Component', () => {
  it('displays count from store', () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            initialState: {
              counter: { count: 42 }
            }
          })
        ]
      }
    })
    
    expect(wrapper.text()).toContain('42')
  })
  
  it('calls increment action on click', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [createTestingPinia()]
      }
    })
    
    const store = useCounterStore()
    
    await wrapper.find('button').trigger('click')
    
    expect(store.increment).toHaveBeenCalled()
  })
})
```

## @pinia/testing Options

```typescript
import { createTestingPinia } from '@pinia/testing'

createTestingPinia({
  // Initial state for stores
  initialState: {
    counter: { count: 10 },
    user: { name: 'Test User' }
  },
  
  // Stub all actions (default: true)
  stubActions: true,
  
  // Create spies for actions (default: true with vitest/jest)
  createSpy: vi.fn,
  
  // Plugins to use
  plugins: [myPlugin]
})
```

## Resources

- [Testing Stores](https://pinia.vuejs.org/cookbook/testing.html) — Official guide to testing Pinia stores

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*