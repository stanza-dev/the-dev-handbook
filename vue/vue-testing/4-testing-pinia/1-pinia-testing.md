---
source_course: "vue-testing"
source_lesson: "vue-testing-pinia-testing"
---

# Testing Pinia Stores

Pinia provides a testing helper that makes it easy to test stores and components that use them.

## Setup

```bash
npm install -D @pinia/testing
```

## Testing Stores Directly

```typescript
import { setActivePinia, createPinia } from 'pinia'
import { useCounterStore } from '@/stores/counter'

describe('Counter Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('has initial state', () => {
    const store = useCounterStore()
    
    expect(store.count).toBe(0)
  })
  
  it('increments count', () => {
    const store = useCounterStore()
    
    store.increment()
    
    expect(store.count).toBe(1)
  })
  
  it('computes double', () => {
    const store = useCounterStore()
    store.count = 5
    
    expect(store.double).toBe(10)
  })
})
```

## Testing Components with Stores

```typescript
import { mount } from '@vue/test-utils'
import { createTestingPinia } from '@pinia/testing'
import { vi } from 'vitest'
import Counter from './Counter.vue'
import { useCounterStore } from '@/stores/counter'

describe('Counter Component', () => {
  it('renders count from store', () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            initialState: {
              counter: { count: 10 }
            }
          })
        ]
      }
    })
    
    expect(wrapper.text()).toContain('10')
  })
  
  it('calls increment action', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [createTestingPinia()]
      }
    })
    
    const store = useCounterStore()
    
    await wrapper.find('button').trigger('click')
    
    // Actions are stubbed by default
    expect(store.increment).toHaveBeenCalled()
  })
})
```

## createTestingPinia Options

```typescript
createTestingPinia({
  // Initial state for stores
  initialState: {
    counter: { count: 100 },
    user: { name: 'Test User' }
  },
  
  // Stub actions (default: true)
  stubActions: true,
  
  // Custom spy function (default: vi.fn)
  createSpy: vi.fn,
  
  // Plugins to use
  plugins: [piniaPluginPersistedstate]
})
```

## Testing Real Actions

```typescript
describe('With real actions', () => {
  it('executes increment', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            stubActions: false  // Run real actions
          })
        ]
      }
    })
    
    const store = useCounterStore()
    expect(store.count).toBe(0)
    
    await wrapper.find('button').trigger('click')
    
    expect(store.count).toBe(1)  // Actually incremented
  })
})
```

## Mocking Store Getters

```typescript
it('shows loading state', () => {
  const wrapper = mount(UserList, {
    global: {
      plugins: [
        createTestingPinia({
          initialState: {
            users: {
              users: [],
              loading: true
            }
          }
        })
      ]
    }
  })
  
  expect(wrapper.find('.loading').exists()).toBe(true)
})
```

## Testing Async Actions

```typescript
import { flushPromises } from '@vue/test-utils'

describe('Async Store', () => {
  it('fetches users', async () => {
    setActivePinia(createPinia())
    const store = useUserStore()
    
    // Mock fetch
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve([{ id: 1, name: 'Alice' }])
    })
    
    await store.fetchUsers()
    
    expect(store.users).toHaveLength(1)
    expect(store.users[0].name).toBe('Alice')
  })
})
```

## Testing Store Subscriptions

```typescript
it('subscribes to changes', () => {
  setActivePinia(createPinia())
  const store = useCounterStore()
  const callback = vi.fn()
  
  store.$subscribe(callback)
  store.increment()
  
  expect(callback).toHaveBeenCalled()
})
```

## Resources

- [Testing Pinia](https://pinia.vuejs.org/cookbook/testing.html) — Official Pinia testing guide

---

> 📘 *This lesson is part of the [Vue Testing Strategies](https://stanza.dev/courses/vue-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*