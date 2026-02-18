---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-ref-vs-reactive-decision"
---

# ref() vs reactive(): Making the Right Choice

Both `ref()` and `reactive()` create reactive state, but they have different use cases. Understanding when to use each is crucial for clean, maintainable code.

## Quick Comparison

| Feature | ref() | reactive() |
|---------|-------|------------|
| Works with primitives | ✅ Yes | ❌ No |
| Needs `.value` in JS | ✅ Yes | ❌ No |
| Can be reassigned | ✅ Yes | ❌ No |
| Destructuring safe | ✅ Yes (keeps reactivity) | ❌ No (loses reactivity) |
| Auto-unwrap in templates | ✅ Yes | ✅ N/A |

## When to Use ref()

### 1. Primitive Values

```typescript
import { ref } from 'vue'

// ✅ Perfect for primitives
const count = ref(0)
const name = ref('Alice')
const isLoading = ref(false)
const selectedId = ref<number | null>(null)
```

### 2. Values That May Be Replaced

```typescript
import { ref } from 'vue'

// ✅ Can replace the entire value
const user = ref<User | null>(null)

async function loadUser() {
  user.value = await fetchUser()  // Complete replacement
}

const items = ref<Item[]>([])

function resetItems() {
  items.value = []  // Replace entire array
}
```

### 3. Return Values from Composables

```typescript
import { ref, computed } from 'vue'

function useCounter() {
  const count = ref(0)
  const double = computed(() => count.value * 2)
  
  function increment() {
    count.value++
  }
  
  // ✅ refs can be destructured by the consumer
  return { count, double, increment }
}

// Consumer can destructure safely
const { count, double, increment } = useCounter()
```

## When to Use reactive()

### 1. Grouped Related State

```typescript
import { reactive } from 'vue'

// ✅ Related properties that change together
const mouse = reactive({
  x: 0,
  y: 0
})

window.addEventListener('mousemove', (e) => {
  mouse.x = e.clientX
  mouse.y = e.clientY
})
```

### 2. Form State

```typescript
import { reactive } from 'vue'

// ✅ Forms with multiple fields
const form = reactive({
  email: '',
  password: '',
  rememberMe: false
})

// Clean access without .value
function validate() {
  return form.email.includes('@') && form.password.length >= 8
}
```

### 3. State That Won't Be Reassigned

```typescript
import { reactive } from 'vue'

// ✅ Configuration objects
const settings = reactive({
  theme: 'dark',
  fontSize: 14,
  notifications: true
})

// Modify properties, never reassign the whole object
settings.theme = 'light'
```

## The Vue Team's Recommendation

The official recommendation is to use **ref() as the primary API** because:

```typescript
import { ref } from 'vue'

// ✅ Consistent: always use .value in JS
const count = ref(0)
const user = ref({ name: 'Alice' })
const items = ref(['a', 'b', 'c'])

// Clear what's reactive vs not
function process(count: number) {  // plain number
  // ...
}

process(count.value)  // Obvious we're unwrapping
```

## Mixing ref() and reactive()

You can use both in the same component:

```typescript
import { ref, reactive, computed } from 'vue'

// Primitives and replaceable values: ref
const isLoading = ref(false)
const error = ref<Error | null>(null)
const selectedUserId = ref<number | null>(null)

// Grouped state: reactive
const filters = reactive({
  search: '',
  status: 'all',
  sortBy: 'name'
})

// Computed from both
const activeFilters = computed(() => {
  return Object.entries(filters)
    .filter(([_, value]) => value !== '' && value !== 'all')
})
```

## Converting Between ref and reactive

### toRef() - Create ref from reactive property

```typescript
import { reactive, toRef } from 'vue'

const state = reactive({ count: 0 })

// Create a ref that syncs with state.count
const countRef = toRef(state, 'count')

countRef.value++  // Also updates state.count
```

### toRefs() - Convert all properties to refs

```typescript
import { reactive, toRefs } from 'vue'

const state = reactive({ count: 0, name: 'Vue' })

// Convert all properties to refs
const { count, name } = toRefs(state)

count.value++    // Updates state.count
name.value = 'Vue 3'  // Updates state.name
```

## Best Practices Summary

1. **Default to ref()** for most cases
2. **Use reactive()** for tightly coupled state objects
3. **Don't mix patterns randomly** - be consistent
4. **Use toRefs()** when destructuring from reactive
5. **Use TypeScript** to catch reactivity mistakes

## Resources

- [Reactivity Fundamentals](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) — Official guide on ref and reactive

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*