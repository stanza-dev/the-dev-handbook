---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-computed-internals"
---

# How Computed Properties Work

Computed properties are one of Vue's most powerful features. They cache derived values and only recompute when their dependencies change.

## The Caching Mechanism

Unlike methods, computed properties cache their results:

```vue
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// Computed: cached, runs once until deps change
const fullName = computed(() => {
  console.log('Computing fullName')  // Only logs when deps change
  return `${firstName.value} ${lastName.value}`
})

// Method: runs every time
function getFullName() {
  console.log('Running getFullName')  // Logs every call
  return `${firstName.value} ${lastName.value}`
}
</script>

<template>
  <!-- Computed: logs once, uses cache for all 3 -->
  <p>{{ fullName }}</p>
  <p>{{ fullName }}</p>
  <p>{{ fullName }}</p>
  
  <!-- Method: logs 3 times -->
  <p>{{ getFullName() }}</p>
  <p>{{ getFullName() }}</p>
  <p>{{ getFullName() }}</p>
</template>
```

## Dependency Tracking

Vue automatically tracks which reactive values a computed property accesses:

```typescript
import { ref, computed } from 'vue'

const price = ref(100)
const quantity = ref(2)
const taxRate = ref(0.1)
const applyDiscount = ref(false)
const discountPercent = ref(0.2)

const total = computed(() => {
  // Vue tracks: price, quantity, taxRate
  let subtotal = price.value * quantity.value
  let tax = subtotal * taxRate.value
  
  // Conditionally tracks: applyDiscount, discountPercent
  if (applyDiscount.value) {
    subtotal -= subtotal * discountPercent.value
  }
  
  return subtotal + tax
})
```

**Key insight**: Dependencies are tracked dynamically. If `applyDiscount` is false, changes to `discountPercent` won't trigger recomputation!

## Computed vs Methods vs Watchers

| Use Case | Use |
|----------|-----|
| Derive data from state | `computed()` |
| Transform data for display | `computed()` |
| Need parameters | Method |
| Perform side effects | `watch()` / `watchEffect()` |
| Expensive calculation used multiple times | `computed()` |

## Chained Computed Properties

Computed properties can depend on other computed properties:

```typescript
import { ref, computed } from 'vue'

const items = ref([
  { name: 'Apple', price: 1.5, quantity: 3 },
  { name: 'Banana', price: 0.75, quantity: 5 },
  { name: 'Orange', price: 2, quantity: 2 }
])

// First computed: individual totals
const itemTotals = computed(() => 
  items.value.map(item => ({
    ...item,
    total: item.price * item.quantity
  }))
)

// Second computed: depends on first
const subtotal = computed(() => 
  itemTotals.value.reduce((sum, item) => sum + item.total, 0)
)

// Third computed: depends on second
const tax = computed(() => subtotal.value * 0.1)

// Fourth computed: depends on second and third
const grandTotal = computed(() => subtotal.value + tax.value)
```

Vue handles the dependency chain automatically!

## Debugging Computed Properties

Use `onTrack` and `onTrigger` for debugging:

```typescript
import { ref, computed } from 'vue'

const count = ref(0)

const double = computed(() => count.value * 2, {
  onTrack(e) {
    // Called when a dependency is tracked
    console.log('Tracking:', e)
  },
  onTrigger(e) {
    // Called when computation is triggered
    console.log('Triggered by:', e)
    debugger  // Pause execution here
  }
})
```

## Performance Considerations

### When Computed Shines

```typescript
// ✅ Good: expensive filter used in multiple places
const filteredItems = computed(() => {
  return largeArray.value
    .filter(item => item.active)
    .sort((a, b) => b.score - a.score)
    .slice(0, 100)
})
```

### When to Avoid

```typescript
// ❌ Unnecessary: simple property access
const userName = computed(() => user.value.name)

// ✅ Just access directly in template
// {{ user.name }}

// ❌ Avoid: side effects in computed
const data = computed(() => {
  console.log('Fetching...')  // Side effect!
  return fetchData()  // Async side effect!
})
```

## Getters and Setters

Computed properties can be writable:

```typescript
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(newValue: string) {
    const parts = newValue.split(' ')
    firstName.value = parts[0] || ''
    lastName.value = parts.slice(1).join(' ') || ''
  }
})

// Usage
fullName.value = 'Jane Smith'  // Sets firstName='Jane', lastName='Smith'
```

## Resources

- [Computed Properties](https://vuejs.org/guide/essentials/computed.html) — Official guide on computed properties

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*