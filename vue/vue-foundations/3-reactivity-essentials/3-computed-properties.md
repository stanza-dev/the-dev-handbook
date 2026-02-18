---
source_course: "vue-foundations"
source_lesson: "vue-foundations-computed-properties"
---

# Computed Properties for Derived State

Computed properties let you declare derived state—values that depend on other reactive values and update automatically when their dependencies change.

## Why Computed Properties?

You could put complex logic directly in templates:

```vue
<template>
  <!-- This works but is messy and repetitive -->
  <p>{{ items.filter(item => item.done).length }} completed</p>
  <p>{{ items.filter(item => item.done).length }} of {{ items.length }}</p>
</template>
```

Computed properties solve this by:
- Moving logic out of templates
- Caching results for performance
- Automatically updating when dependencies change

## Basic Usage

```vue
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// Computed property - automatically updates when firstName or lastName changes
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})
</script>

<template>
  <p>First: {{ firstName }}</p>
  <p>Last: {{ lastName }}</p>
  <p>Full: {{ fullName }}</p>
  
  <input v-model="firstName" placeholder="First name" />
  <input v-model="lastName" placeholder="Last name" />
</template>
```

When you type in either input, `fullName` automatically updates!

## Computed vs Methods

### Method Approach

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

// Method - called every time the template renders
function doubleCount() {
  console.log('Method called')  // Logs on EVERY render
  return count.value * 2
}
</script>

<template>
  <p>{{ doubleCount() }}</p>
  <p>{{ doubleCount() }}</p>
  <p>{{ doubleCount() }}</p>
  <!-- Method runs 3 times per render! -->
</template>
```

### Computed Approach

```vue
<script setup>
import { ref, computed } from 'vue'

const count = ref(0)

// Computed - cached based on dependencies
const doubleCount = computed(() => {
  console.log('Computed evaluated')  // Logs only when count changes
  return count.value * 2
})
</script>

<template>
  <p>{{ doubleCount }}</p>
  <p>{{ doubleCount }}</p>
  <p>{{ doubleCount }}</p>
  <!-- Same cached value used 3 times -->
</template>
```

**Key difference**: Computed properties are **cached** and only re-evaluate when their reactive dependencies change.

## Practical Examples

### Filtering a List

```vue
<script setup>
import { ref, computed } from 'vue'

const todos = ref([
  { id: 1, text: 'Learn Vue', done: true },
  { id: 2, text: 'Build an app', done: false },
  { id: 3, text: 'Deploy', done: false }
])

const filter = ref('all')  // 'all', 'active', 'completed'

const filteredTodos = computed(() => {
  switch (filter.value) {
    case 'active':
      return todos.value.filter(t => !t.done)
    case 'completed':
      return todos.value.filter(t => t.done)
    default:
      return todos.value
  }
})

const stats = computed(() => ({
  total: todos.value.length,
  completed: todos.value.filter(t => t.done).length,
  remaining: todos.value.filter(t => !t.done).length
}))
</script>

<template>
  <div>
    <button @click="filter = 'all'">All ({{ stats.total }})</button>
    <button @click="filter = 'active'">Active ({{ stats.remaining }})</button>
    <button @click="filter = 'completed'">Completed ({{ stats.completed }})</button>
  </div>
  
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      {{ todo.text }}
    </li>
  </ul>
</template>
```

### Sorting Data

```vue
<script setup>
import { ref, computed } from 'vue'

const products = ref([
  { id: 1, name: 'Laptop', price: 999 },
  { id: 2, name: 'Phone', price: 699 },
  { id: 3, name: 'Tablet', price: 499 }
])

const sortBy = ref('name')  // 'name', 'price'
const sortOrder = ref('asc')  // 'asc', 'desc'

const sortedProducts = computed(() => {
  const sorted = [...products.value].sort((a, b) => {
    const aVal = a[sortBy.value]
    const bVal = b[sortBy.value]
    
    if (typeof aVal === 'string') {
      return aVal.localeCompare(bVal)
    }
    return aVal - bVal
  })
  
  return sortOrder.value === 'desc' ? sorted.reverse() : sorted
})
</script>
```

### Form Validation

```vue
<script setup>
import { ref, computed } from 'vue'

const email = ref('')
const password = ref('')
const confirmPassword = ref('')

const errors = computed(() => {
  const errs = []
  
  if (!email.value) {
    errs.push('Email is required')
  } else if (!/^[^@]+@[^@]+\.[^@]+$/.test(email.value)) {
    errs.push('Invalid email format')
  }
  
  if (password.value.length < 8) {
    errs.push('Password must be at least 8 characters')
  }
  
  if (password.value !== confirmPassword.value) {
    errs.push('Passwords do not match')
  }
  
  return errs
})

const isValid = computed(() => errors.value.length === 0)
</script>

<template>
  <form @submit.prevent>
    <input v-model="email" type="email" placeholder="Email" />
    <input v-model="password" type="password" placeholder="Password" />
    <input v-model="confirmPassword" type="password" placeholder="Confirm" />
    
    <ul v-if="errors.length" class="errors">
      <li v-for="error in errors" :key="error">{{ error }}</li>
    </ul>
    
    <button :disabled="!isValid">Submit</button>
  </form>
</template>
```

## Writable Computed Properties

Computed properties are read-only by default. For writable computed, provide getter and setter:

```vue
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(newValue) {
    const parts = newValue.split(' ')
    firstName.value = parts[0] || ''
    lastName.value = parts.slice(1).join(' ') || ''
  }
})
</script>

<template>
  <!-- Two-way binding works! -->
  <input v-model="fullName" />
  <p>First: {{ firstName }}</p>
  <p>Last: {{ lastName }}</p>
</template>
```

## Resources

- [Computed Properties](https://vuejs.org/guide/essentials/computed.html) — Official Vue documentation on computed properties

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*