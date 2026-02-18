---
source_course: "vue-pinia"
source_lesson: "vue-pinia-getters-computed"
---

# Getters and Computed State

Getters in Pinia are the equivalent of computed properties. They derive values from state and are cached until their dependencies change.

## Basic Getters

### Options Syntax

```typescript
import { defineStore } from 'pinia'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [] as CartItem[],
    taxRate: 0.1
  }),
  
  getters: {
    // Simple getter with state parameter
    itemCount: (state) => state.items.length,
    
    // Getter returning a value
    subtotal: (state) => 
      state.items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    
    // Using this to access other getters
    tax(): number {
      return this.subtotal * this.taxRate
    },
    
    // Using other getters
    total(): number {
      return this.subtotal + this.tax
    },
    
    // Getter that returns a function (for parameters)
    getItemById: (state) => {
      return (id: number) => state.items.find(item => item.id === id)
    }
  }
})
```

### Setup Syntax

```typescript
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useCartStore = defineStore('cart', () => {
  const items = ref<CartItem[]>([])
  const taxRate = ref(0.1)
  
  // Getters as computed
  const itemCount = computed(() => items.value.length)
  
  const subtotal = computed(() => 
    items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
  )
  
  const tax = computed(() => subtotal.value * taxRate.value)
  
  const total = computed(() => subtotal.value + tax.value)
  
  // Function getter
  function getItemById(id: number) {
    return items.value.find(item => item.id === id)
  }
  
  return {
    items,
    taxRate,
    itemCount,
    subtotal,
    tax,
    total,
    getItemById
  }
})
```

## Accessing Other Stores in Getters

```typescript
import { defineStore } from 'pinia'
import { useUserStore } from './user'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [] as CartItem[]
  }),
  
  getters: {
    // Access another store
    discountedTotal(): number {
      const userStore = useUserStore()
      const discount = userStore.isPremium ? 0.1 : 0
      return this.total * (1 - discount)
    }
  }
})
```

## Parameterized Getters

Getters can't take parameters directly, but can return functions:

```typescript
export const useProductStore = defineStore('products', {
  state: () => ({
    products: [] as Product[]
  }),
  
  getters: {
    // Returns a function that takes parameters
    getByCategory: (state) => {
      return (category: string) => 
        state.products.filter(p => p.category === category)
    },
    
    // Price range filter
    getByPriceRange: (state) => {
      return (min: number, max: number) =>
        state.products.filter(p => p.price >= min && p.price <= max)
    },
    
    // Search
    search: (state) => {
      return (query: string) => {
        const lower = query.toLowerCase()
        return state.products.filter(p => 
          p.name.toLowerCase().includes(lower) ||
          p.description.toLowerCase().includes(lower)
        )
      }
    }
  }
})

// Usage
const store = useProductStore()
const electronics = store.getByCategory('electronics')
const affordable = store.getByPriceRange(0, 50)
const results = store.search('laptop')
```

**Note**: Parameterized getters are NOT cached!

## Getter Caching

```typescript
getters: {
  // ✅ Cached - recalculates only when items change
  itemCount: (state) => {
    console.log('Computing itemCount')  // Logs once per dependency change
    return state.items.length
  },
  
  // ❌ NOT cached - runs on every call
  getExpensiveItems: (state) => (minPrice: number) => {
    console.log('Filtering items')  // Logs on every call
    return state.items.filter(i => i.price > minPrice)
  }
}
```

## Complex Getter Example

```typescript
export const useAnalyticsStore = defineStore('analytics', {
  state: () => ({
    orders: [] as Order[]
  }),
  
  getters: {
    // Group by month
    ordersByMonth(): Record<string, Order[]> {
      return this.orders.reduce((acc, order) => {
        const month = order.date.toISOString().slice(0, 7)
        if (!acc[month]) acc[month] = []
        acc[month].push(order)
        return acc
      }, {} as Record<string, Order[]>)
    },
    
    // Monthly totals
    monthlyTotals(): Record<string, number> {
      const result: Record<string, number> = {}
      for (const [month, orders] of Object.entries(this.ordersByMonth)) {
        result[month] = orders.reduce((sum, o) => sum + o.total, 0)
      }
      return result
    },
    
    // Best performing month
    bestMonth(): { month: string; total: number } | null {
      const entries = Object.entries(this.monthlyTotals)
      if (entries.length === 0) return null
      
      return entries.reduce((best, [month, total]) => 
        total > best.total ? { month, total } : best
      , { month: '', total: 0 })
    }
  }
})
```

## Resources

- [Getters](https://pinia.vuejs.org/core-concepts/getters.html) — Official documentation on Pinia getters

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*