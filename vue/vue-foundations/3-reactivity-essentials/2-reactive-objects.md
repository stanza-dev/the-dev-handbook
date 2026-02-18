---
source_course: "vue-foundations"
source_lesson: "vue-foundations-reactive-objects"
---

# Reactive Objects with reactive()

While `ref()` wraps any value, `reactive()` is specifically designed for objects. It makes the entire object deeply reactive without needing `.value`.

## Basic Usage

```vue
<script setup>
import { reactive } from 'vue'

const state = reactive({
  count: 0,
  message: 'Hello',
  user: {
    name: 'Alice',
    email: 'alice@example.com'
  }
})

// No .value needed!
state.count++
state.message = 'Hello Vue!'
state.user.name = 'Bob'
</script>

<template>
  <p>Count: {{ state.count }}</p>
  <p>Message: {{ state.message }}</p>
  <p>User: {{ state.user.name }}</p>
</template>
```

## reactive() vs ref() - When to Use Which?

### Use ref() when:

```vue
<script setup>
import { ref } from 'vue'

// ✅ Primitive values
const count = ref(0)
const isLoading = ref(false)
const name = ref('Alice')

// ✅ Values that might be reassigned entirely
const selectedUser = ref(null)
selectedUser.value = { id: 1, name: 'Bob' }  // Works!

// ✅ Arrays you might replace
const items = ref(['A', 'B'])
items.value = ['X', 'Y', 'Z']  // Works!
</script>
```

### Use reactive() when:

```vue
<script setup>
import { reactive } from 'vue'

// ✅ Complex state objects
const form = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
})

// ✅ State that won't be reassigned
const settings = reactive({
  theme: 'dark',
  language: 'en',
  notifications: {
    email: true,
    push: false
  }
})
</script>
```

## Limitations of reactive()

### 1. Only Works with Objects

```javascript
import { reactive } from 'vue'

// ❌ These don't work
const count = reactive(0)        // Returns 0, not reactive
const message = reactive('hi')   // Returns 'hi', not reactive

// ✅ Must be an object
const state = reactive({ count: 0 })
```

### 2. Cannot Reassign the Entire Object

```javascript
import { reactive } from 'vue'

let state = reactive({ count: 0 })

// ❌ This breaks reactivity!
state = reactive({ count: 1 })

// ✅ Instead, modify properties
state.count = 1

// Or use Object.assign
Object.assign(state, { count: 1, message: 'new' })
```

### 3. Destructuring Breaks Reactivity

```javascript
import { reactive } from 'vue'

const state = reactive({ count: 0, name: 'Alice' })

// ❌ These lose reactivity!
const { count, name } = state
count++  // Not reactive, won't update the UI

// ✅ Use toRefs() if you need to destructure
import { toRefs } from 'vue'
const { count, name } = toRefs(state)
count.value++  // Reactive!
```

## Deep Reactivity

Both `ref()` and `reactive()` create deeply reactive objects:

```vue
<script setup>
import { reactive } from 'vue'

const state = reactive({
  level1: {
    level2: {
      level3: {
        value: 'deep'
      }
    }
  }
})

// All levels are reactive
state.level1.level2.level3.value = 'changed'  // Triggers update!
</script>
```

## Combining ref() and reactive()

You can mix both approaches:

```vue
<script setup>
import { ref, reactive } from 'vue'

// Simple values with ref
const isLoading = ref(false)
const error = ref(null)

// Complex state with reactive
const form = reactive({
  email: '',
  password: ''
})

const user = reactive({
  profile: null,
  preferences: {
    theme: 'light'
  }
})

async function login() {
  isLoading.value = true
  error.value = null
  
  try {
    const response = await api.login(form.email, form.password)
    user.profile = response.user
  } catch (e) {
    error.value = e.message
  } finally {
    isLoading.value = false
  }
}
</script>
```

## The Recommendation

The Vue team recommends using `ref()` as the primary reactive API because:

1. **Consistent API**: Always use `.value` in JavaScript
2. **More flexible**: Works with any value type
3. **No reassignment issues**: Can replace entire values
4. **Better destructuring**: Works naturally with function returns

```vue
<script setup>
import { ref } from 'vue'

// This pattern works well for most cases
const count = ref(0)
const user = ref(null)
const items = ref([])

const form = ref({
  email: '',
  password: ''
})
</script>
```

## Resources

- [reactive() API Reference](https://vuejs.org/api/reactivity-core.html#reactive) — Official API documentation for reactive()

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*