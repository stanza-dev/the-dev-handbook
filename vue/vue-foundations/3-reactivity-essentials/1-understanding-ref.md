---
source_course: "vue-foundations"
source_lesson: "vue-foundations-understanding-ref"
---

# Understanding ref() for Reactive State

Reactivity is the heart of Vue. When you change your data, the UI automatically updates. The `ref()` function is the primary way to create reactive state in the Composition API.

## What is ref()?

`ref()` takes a value and returns a reactive, mutable object with a single property `.value` that points to your data:

```vue
<script setup>
import { ref } from 'vue'

// Create reactive state
const count = ref(0)
const message = ref('Hello Vue!')
const isActive = ref(true)
const user = ref({ name: 'Alice', age: 25 })

// Access and modify with .value
console.log(count.value)  // 0
count.value++
console.log(count.value)  // 1

message.value = 'Hello World!'
isActive.value = false
user.value.name = 'Bob'
</script>
```

## Why .value?

JavaScript primitives (numbers, strings, booleans) are passed by value, not reference. Vue can't track changes to plain variables:

```javascript
// This doesn't work - Vue can't track it
let count = 0
count++  // Vue has no idea this happened

// This works - Vue tracks the ref object
const count = ref(0)
count.value++  // Vue detects the change
```

The `.value` property lets Vue intercept access and track dependencies.

## Automatic Unwrapping in Templates

Good news: you don't need `.value` in templates! Vue automatically unwraps refs:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
const message = ref('Hello')
</script>

<template>
  <!-- No .value needed! -->
  <p>Count: {{ count }}</p>
  <p>Message: {{ message }}</p>
  
  <!-- Also works in attributes -->
  <input :value="message" />
  
  <!-- And in expressions -->
  <p>Double: {{ count * 2 }}</p>
</template>
```

## Working with Different Types

### Primitive Values

```vue
<script setup>
import { ref } from 'vue'

// Numbers
const price = ref(29.99)
const quantity = ref(1)

// Strings
const firstName = ref('John')
const lastName = ref('Doe')

// Booleans
const isLoading = ref(false)
const hasError = ref(false)

// Null/Undefined
const selectedItem = ref(null)
const optionalValue = ref(undefined)
</script>
```

### Objects and Arrays

`ref()` works with objects and arrays too—they become deeply reactive:

```vue
<script setup>
import { ref } from 'vue'

const user = ref({
  name: 'Alice',
  email: 'alice@example.com',
  settings: {
    theme: 'dark',
    notifications: true
  }
})

const items = ref(['Apple', 'Banana', 'Cherry'])

// All these trigger reactivity:
user.value.name = 'Bob'
user.value.settings.theme = 'light'
items.value.push('Date')
items.value[0] = 'Apricot'
</script>
```

## TypeScript with ref()

TypeScript infers the type automatically, or you can be explicit:

```typescript
import { ref } from 'vue'

// Type inference
const count = ref(0)           // Ref<number>
const message = ref('hello')   // Ref<string>

// Explicit typing
const id = ref<number | null>(null)
const items = ref<string[]>([])

type User = {
  id: number
  name: string
  email: string
}

const user = ref<User | null>(null)

// Later:
user.value = {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com'
}
```

## Common Patterns

### Toggle Boolean

```vue
<script setup>
import { ref } from 'vue'

const isOpen = ref(false)

function toggle() {
  isOpen.value = !isOpen.value
}
</script>

<template>
  <button @click="toggle">
    {{ isOpen ? 'Close' : 'Open' }}
  </button>
  <div v-if="isOpen">Content here</div>
</template>
```

### Counter

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}

function decrement() {
  count.value--
}

function reset() {
  count.value = 0
}
</script>

<template>
  <p>Count: {{ count }}</p>
  <button @click="decrement">-</button>
  <button @click="increment">+</button>
  <button @click="reset">Reset</button>
</template>
```

### Form Input

```vue
<script setup>
import { ref } from 'vue'

const searchQuery = ref('')
const selectedOption = ref('all')

function handleSearch() {
  console.log('Searching for:', searchQuery.value)
}
</script>

<template>
  <input 
    v-model="searchQuery" 
    placeholder="Search..." 
  />
  <select v-model="selectedOption">
    <option value="all">All</option>
    <option value="active">Active</option>
    <option value="completed">Completed</option>
  </select>
</template>
```

## Resources

- [Reactivity Fundamentals](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) — Official Vue documentation on ref() and reactivity basics

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*