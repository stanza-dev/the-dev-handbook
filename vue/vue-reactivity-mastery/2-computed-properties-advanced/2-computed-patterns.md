---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-computed-patterns"
---

# Advanced Computed Patterns

Let's explore sophisticated patterns for using computed properties effectively.

## Filtering and Sorting

```typescript
import { ref, computed } from 'vue'

type Product = {
  id: number
  name: string
  price: number
  category: string
  inStock: boolean
}

const products = ref<Product[]>([/* ... */])

// Filter state
const searchQuery = ref('')
const selectedCategory = ref('all')
const showOnlyInStock = ref(false)
const sortBy = ref<'name' | 'price'>('name')
const sortOrder = ref<'asc' | 'desc'>('asc')

// Computed: filtered and sorted
const filteredProducts = computed(() => {
  let result = products.value
  
  // Filter by search
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(p => 
      p.name.toLowerCase().includes(query)
    )
  }
  
  // Filter by category
  if (selectedCategory.value !== 'all') {
    result = result.filter(p => 
      p.category === selectedCategory.value
    )
  }
  
  // Filter by stock
  if (showOnlyInStock.value) {
    result = result.filter(p => p.inStock)
  }
  
  // Sort
  result = [...result].sort((a, b) => {
    const aVal = a[sortBy.value]
    const bVal = b[sortBy.value]
    const modifier = sortOrder.value === 'asc' ? 1 : -1
    
    if (typeof aVal === 'string') {
      return aVal.localeCompare(bVal as string) * modifier
    }
    return ((aVal as number) - (bVal as number)) * modifier
  })
  
  return result
})

// Derived stats
const stats = computed(() => ({
  total: products.value.length,
  filtered: filteredProducts.value.length,
  inStock: filteredProducts.value.filter(p => p.inStock).length,
  avgPrice: filteredProducts.value.reduce((sum, p) => sum + p.price, 0) / 
            filteredProducts.value.length || 0
}))
```

## Memoized Expensive Operations

```typescript
import { ref, computed } from 'vue'

const points = ref<[number, number][]>([/* thousands of points */])
const tolerance = ref(0.1)

// Expensive calculation - only runs when points or tolerance change
const simplifiedPath = computed(() => {
  console.time('simplify')
  const result = douglasPeuckerSimplify(points.value, tolerance.value)
  console.timeEnd('simplify')
  return result
})

// Further derived values are cheap
const pathLength = computed(() => 
  calculatePathLength(simplifiedPath.value)
)

const boundingBox = computed(() => 
  calculateBounds(simplifiedPath.value)
)
```

## Form Validation

```typescript
import { reactive, computed } from 'vue'

const form = reactive({
  email: '',
  password: '',
  confirmPassword: '',
  age: ''
})

const validation = computed(() => {
  const errors: Record<string, string> = {}
  
  // Email
  if (!form.email) {
    errors.email = 'Email is required'
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
    errors.email = 'Invalid email format'
  }
  
  // Password
  if (!form.password) {
    errors.password = 'Password is required'
  } else if (form.password.length < 8) {
    errors.password = 'Password must be at least 8 characters'
  } else if (!/[A-Z]/.test(form.password)) {
    errors.password = 'Password must contain an uppercase letter'
  }
  
  // Confirm password
  if (form.password !== form.confirmPassword) {
    errors.confirmPassword = 'Passwords do not match'
  }
  
  // Age
  const age = parseInt(form.age)
  if (isNaN(age) || age < 18 || age > 120) {
    errors.age = 'Please enter a valid age (18-120)'
  }
  
  return errors
})

const isValid = computed(() => 
  Object.keys(validation.value).length === 0
)

const firstError = computed(() => 
  Object.values(validation.value)[0] || null
)
```

## Computed with External Data

```typescript
import { ref, computed } from 'vue'

const exchangeRates = ref<Record<string, number>>({
  USD: 1,
  EUR: 0.85,
  GBP: 0.73
})

const amounts = ref([
  { currency: 'USD', value: 100 },
  { currency: 'EUR', value: 200 },
  { currency: 'GBP', value: 150 }
])

const targetCurrency = ref('USD')

const convertedAmounts = computed(() => {
  const targetRate = exchangeRates.value[targetCurrency.value]
  
  return amounts.value.map(amount => {
    const sourceRate = exchangeRates.value[amount.currency]
    const inUSD = amount.value / sourceRate
    const converted = inUSD * targetRate
    
    return {
      ...amount,
      converted: Math.round(converted * 100) / 100,
      targetCurrency: targetCurrency.value
    }
  })
})

const totalConverted = computed(() => 
  convertedAmounts.value.reduce((sum, a) => sum + a.converted, 0)
)
```

## Pagination

```typescript
import { ref, computed } from 'vue'

const allItems = ref<Item[]>([/* many items */])
const currentPage = ref(1)
const itemsPerPage = ref(10)

const totalPages = computed(() => 
  Math.ceil(allItems.value.length / itemsPerPage.value)
)

const paginatedItems = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value
  const end = start + itemsPerPage.value
  return allItems.value.slice(start, end)
})

const pageNumbers = computed(() => {
  const pages: number[] = []
  const total = totalPages.value
  const current = currentPage.value
  
  // Show first, last, and surrounding pages
  for (let i = 1; i <= total; i++) {
    if (
      i === 1 || 
      i === total || 
      (i >= current - 2 && i <= current + 2)
    ) {
      pages.push(i)
    }
  }
  
  return pages
})

const canGoPrevious = computed(() => currentPage.value > 1)
const canGoNext = computed(() => currentPage.value < totalPages.value)
```

## Resources

- [Computed Best Practices](https://vuejs.org/guide/essentials/computed.html#best-practices) — Official best practices for computed properties

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*