---
source_course: "vue-foundations"
source_lesson: "vue-foundations-list-rendering"
---

# List Rendering with v-for

The `v-for` directive renders a list of items based on an array or object. It's one of the most commonly used directives in Vue applications.

## Basic Array Rendering

Use `v-for` with the syntax `item in items`:

```vue
<script setup>
import { ref } from 'vue'

const fruits = ref(['Apple', 'Banana', 'Cherry', 'Date'])
</script>

<template>
  <ul>
    <li v-for="fruit in fruits" :key="fruit">
      {{ fruit }}
    </li>
  </ul>
</template>
```

## Accessing the Index

Get the current index using a second parameter:

```vue
<script setup>
import { ref } from 'vue'

const tasks = ref([
  { id: 1, text: 'Learn Vue', done: true },
  { id: 2, text: 'Build an app', done: false },
  { id: 3, text: 'Deploy to production', done: false }
])
</script>

<template>
  <ul>
    <li v-for="(task, index) in tasks" :key="task.id">
      {{ index + 1 }}. {{ task.text }}
      <span v-if="task.done">✓</span>
    </li>
  </ul>
</template>
```

## The `key` Attribute - Critical!

**Always provide a unique `key`** when using `v-for`. This helps Vue track each element and efficiently update the DOM:

```vue
<script setup>
import { ref } from 'vue'

type User = {
  id: number
  name: string
  email: string
}

const users = ref<User[]>([
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
  { id: 3, name: 'Charlie', email: 'charlie@example.com' }
])
</script>

<template>
  <!-- GOOD: unique id as key -->
  <div v-for="user in users" :key="user.id">
    {{ user.name }} - {{ user.email }}
  </div>
  
  <!-- BAD: index as key (causes issues with reordering) -->
  <div v-for="(user, index) in users" :key="index">
    {{ user.name }}
  </div>
</template>
```

### Why Keys Matter

Without proper keys, Vue may:
- Reuse DOM elements incorrectly when items are reordered
- Lose form input values when the list changes
- Cause visual glitches and performance issues

## Iterating Over Objects

Loop through object properties:

```vue
<script setup>
import { reactive } from 'vue'

const profile = reactive({
  name: 'John Doe',
  email: 'john@example.com',
  role: 'Developer',
  location: 'New York'
})
</script>

<template>
  <!-- Value only -->
  <ul>
    <li v-for="value in profile" :key="value">
      {{ value }}
    </li>
  </ul>
  
  <!-- Value and key -->
  <ul>
    <li v-for="(value, key) in profile" :key="key">
      <strong>{{ key }}:</strong> {{ value }}
    </li>
  </ul>
  
  <!-- Value, key, and index -->
  <ul>
    <li v-for="(value, key, index) in profile" :key="key">
      {{ index + 1 }}. {{ key }}: {{ value }}
    </li>
  </ul>
</template>
```

## Range v-for

Iterate over a range of numbers:

```vue
<template>
  <!-- Renders 1, 2, 3, 4, 5 -->
  <span v-for="n in 5" :key="n">{{ n }}</span>
  
  <!-- Create 10 placeholder items -->
  <div v-for="i in 10" :key="i" class="skeleton-item">
    Loading item {{ i }}...
  </div>
</template>
```

Note: The range starts at 1, not 0.

## v-for on `<template>`

Render multiple elements per iteration without extra wrappers:

```vue
<script setup>
import { ref } from 'vue'

const items = ref([
  { id: 1, title: 'Item 1', description: 'Description 1' },
  { id: 2, title: 'Item 2', description: 'Description 2' }
])
</script>

<template>
  <dl>
    <template v-for="item in items" :key="item.id">
      <dt>{{ item.title }}</dt>
      <dd>{{ item.description }}</dd>
    </template>
  </dl>
</template>
```

## Nested v-for

Loop within loops for nested data:

```vue
<script setup>
import { ref } from 'vue'

const categories = ref([
  {
    id: 1,
    name: 'Electronics',
    products: [
      { id: 101, name: 'Phone' },
      { id: 102, name: 'Laptop' }
    ]
  },
  {
    id: 2,
    name: 'Clothing',
    products: [
      { id: 201, name: 'T-Shirt' },
      { id: 202, name: 'Jeans' }
    ]
  }
])
</script>

<template>
  <div v-for="category in categories" :key="category.id">
    <h3>{{ category.name }}</h3>
    <ul>
      <li v-for="product in category.products" :key="product.id">
        {{ product.name }}
      </li>
    </ul>
  </div>
</template>
```

## Array Change Detection

Vue detects changes to reactive arrays and updates the view:

```vue
<script setup>
import { ref } from 'vue'

const items = ref(['A', 'B', 'C'])

function addItem() {
  items.value.push('New Item')  // Reactive!
}

function removeFirst() {
  items.value.shift()  // Reactive!
}

function replaceAll() {
  items.value = ['X', 'Y', 'Z']  // Reactive!
}
</script>
```

### Mutation Methods (Trigger Updates)

- `push()`, `pop()`
- `shift()`, `unshift()`
- `splice()`
- `sort()`, `reverse()`

### Replacing the Array

Methods like `filter()`, `map()`, `slice()` return new arrays. Assign the result back:

```vue
<script setup>
import { ref } from 'vue'

const numbers = ref([1, 2, 3, 4, 5])

function filterEven() {
  numbers.value = numbers.value.filter(n => n % 2 === 0)
}
</script>
```

## Resources

- [List Rendering](https://vuejs.org/guide/essentials/list.html) — Official Vue documentation on v-for and list rendering

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*