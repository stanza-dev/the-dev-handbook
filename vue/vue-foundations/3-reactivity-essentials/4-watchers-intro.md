---
source_course: "vue-foundations"
source_lesson: "vue-foundations-watchers-intro"
---

# Introduction to Watchers

While computed properties handle derived state, watchers let you perform **side effects** in response to state changes—like fetching data, logging, or modifying the DOM.

## When to Use Watchers

Use watchers when you need to:
- Fetch data when a value changes
- Perform async operations
- Log or debug state changes
- Interact with external systems

## Basic watch()

The `watch()` function takes a source and a callback:

```vue
<script setup>
import { ref, watch } from 'vue'

const searchQuery = ref('')

// Watch a single ref
watch(searchQuery, (newValue, oldValue) => {
  console.log(`Search changed from "${oldValue}" to "${newValue}"`)
})
</script>

<template>
  <input v-model="searchQuery" placeholder="Search..." />
</template>
```

## Watching Different Sources

### Single Ref

```vue
<script setup>
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newVal, oldVal) => {
  console.log(`Count: ${oldVal} → ${newVal}`)
})
</script>
```

### Getter Function

For reactive object properties or computed expressions:

```vue
<script setup>
import { reactive, watch } from 'vue'

const user = reactive({ name: 'Alice', age: 25 })

// Watch a specific property
watch(
  () => user.name,
  (newName) => {
    console.log(`Name changed to: ${newName}`)
  }
)

// Watch a computed expression
watch(
  () => user.age >= 18,
  (isAdult) => {
    console.log(`User is ${isAdult ? 'an adult' : 'a minor'}`)
  }
)
</script>
```

### Multiple Sources

Watch multiple values at once:

```vue
<script setup>
import { ref, watch } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

watch(
  [firstName, lastName],
  ([newFirst, newLast], [oldFirst, oldLast]) => {
    console.log(`Name: ${oldFirst} ${oldLast} → ${newFirst} ${newLast}`)
  }
)
</script>
```

## watchEffect()

`watchEffect()` automatically tracks all reactive dependencies used inside it:

```vue
<script setup>
import { ref, watchEffect } from 'vue'

const count = ref(0)
const multiplier = ref(2)

// Automatically tracks count AND multiplier
watchEffect(() => {
  console.log(`Result: ${count.value * multiplier.value}`)
})
// Logs immediately: "Result: 0"
// Logs again whenever count or multiplier changes
</script>
```

### watch() vs watchEffect()

| Feature | watch() | watchEffect() |
|---------|---------|---------------|
| Explicit dependencies | Yes | No (auto-tracked) |
| Access to old value | Yes | No |
| Lazy by default | Yes | No (runs immediately) |
| Best for | Specific state changes | Multiple dependencies |

## Practical Examples

### Fetching Data on Change

```vue
<script setup>
import { ref, watch } from 'vue'

const userId = ref(1)
const userData = ref(null)
const isLoading = ref(false)

watch(userId, async (newId) => {
  isLoading.value = true
  try {
    const response = await fetch(`/api/users/${newId}`)
    userData.value = await response.json()
  } catch (error) {
    console.error('Failed to fetch user:', error)
  } finally {
    isLoading.value = false
  }
})
</script>

<template>
  <select v-model="userId">
    <option :value="1">User 1</option>
    <option :value="2">User 2</option>
    <option :value="3">User 3</option>
  </select>
  
  <div v-if="isLoading">Loading...</div>
  <div v-else-if="userData">
    <h2>{{ userData.name }}</h2>
    <p>{{ userData.email }}</p>
  </div>
</template>
```

### Debounced Search

```vue
<script setup>
import { ref, watch } from 'vue'

const searchQuery = ref('')
const searchResults = ref([])

let debounceTimeout

watch(searchQuery, (query) => {
  // Clear previous timeout
  clearTimeout(debounceTimeout)
  
  if (!query) {
    searchResults.value = []
    return
  }
  
  // Wait 300ms before searching
  debounceTimeout = setTimeout(async () => {
    const response = await fetch(`/api/search?q=${query}`)
    searchResults.value = await response.json()
  }, 300)
})
</script>
```

### Persisting to LocalStorage

```vue
<script setup>
import { ref, watch } from 'vue'

// Load initial value from localStorage
const theme = ref(localStorage.getItem('theme') || 'light')

// Save to localStorage whenever it changes
watch(theme, (newTheme) => {
  localStorage.setItem('theme', newTheme)
  document.documentElement.setAttribute('data-theme', newTheme)
})
</script>

<template>
  <select v-model="theme">
    <option value="light">Light</option>
    <option value="dark">Dark</option>
    <option value="system">System</option>
  </select>
</template>
```

## Watcher Options

### immediate: Run on Creation

```vue
<script setup>
import { ref, watch } from 'vue'

const userId = ref(1)

watch(
  userId,
  async (id) => {
    await fetchUser(id)
  },
  { immediate: true }  // Fetch immediately, not just on change
)
</script>
```

### deep: Watch Nested Changes

```vue
<script setup>
import { ref, watch } from 'vue'

const settings = ref({
  notifications: {
    email: true,
    push: false
  }
})

watch(
  settings,
  (newSettings) => {
    console.log('Settings changed:', newSettings)
  },
  { deep: true }  // Detect nested property changes
)
</script>
```

## Resources

- [Watchers](https://vuejs.org/guide/essentials/watchers.html) — Official Vue documentation on watch() and watchEffect()

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*