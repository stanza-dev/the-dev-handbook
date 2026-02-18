---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-watcher-options"
---

# Watcher Options and Advanced Patterns

Let's explore the full range of watcher options and patterns for real-world scenarios.

## Watcher Options

### immediate: Run on Setup

```typescript
const userId = ref(1)

// Without immediate: waits for first change
watch(userId, (id) => {
  fetchUser(id)
})
// User isn't fetched until userId changes!

// With immediate: runs immediately
watch(
  userId,
  (id) => {
    fetchUser(id)
  },
  { immediate: true }
)
// User is fetched right away, and on every change
```

### deep: Watch Nested Changes

```typescript
const user = ref({
  profile: {
    name: 'Alice',
    settings: {
      theme: 'dark'
    }
  }
})

// Shallow: only triggers on user.value = newObject
watch(user, () => {
  console.log('User changed')
})

// Deep: triggers on ANY nested change
watch(
  user,
  () => {
    console.log('User or nested property changed')
  },
  { deep: true }
)

user.value.profile.settings.theme = 'light'  // Triggers with deep: true
```

**Note**: `reactive()` objects are automatically deep-watched:

```typescript
const state = reactive({ nested: { value: 1 } })

// Automatically deep
watch(state, () => {
  console.log('State changed')
})

state.nested.value = 2  // Triggers!
```

### once: Run Only Once (Vue 3.4+)

```typescript
watch(
  source,
  (value) => {
    // This only runs once, on first change
    initializeWithValue(value)
  },
  { once: true }
)
```

## Practical Patterns

### Debounced API Calls

```typescript
import { ref, watch } from 'vue'

const searchQuery = ref('')
const results = ref([])
const isSearching = ref(false)

let debounceTimer: number | null = null

watch(searchQuery, (query) => {
  // Clear previous timer
  if (debounceTimer) {
    clearTimeout(debounceTimer)
  }
  
  // Empty query: clear results
  if (!query.trim()) {
    results.value = []
    return
  }
  
  // Debounce: wait 300ms
  debounceTimer = setTimeout(async () => {
    isSearching.value = true
    try {
      const response = await fetch(`/api/search?q=${query}`)
      results.value = await response.json()
    } finally {
      isSearching.value = false
    }
  }, 300)
})
```

### LocalStorage Sync

```typescript
import { ref, watch } from 'vue'

function useLocalStorage<T>(key: string, defaultValue: T) {
  // Initialize from localStorage
  const stored = localStorage.getItem(key)
  const value = ref<T>(
    stored ? JSON.parse(stored) : defaultValue
  )
  
  // Sync to localStorage on changes
  watch(
    value,
    (newValue) => {
      localStorage.setItem(key, JSON.stringify(newValue))
    },
    { deep: true }
  )
  
  return value
}

// Usage
const settings = useLocalStorage('settings', {
  theme: 'light',
  fontSize: 14
})
```

### Route Parameter Watching

```typescript
import { watch } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const post = ref(null)

// Watch route params
watch(
  () => route.params.id,
  async (newId) => {
    if (newId) {
      post.value = await fetchPost(newId)
    }
  },
  { immediate: true }  // Fetch on component mount
)
```

### Conditional Watching

```typescript
import { ref, watch } from 'vue'

const isEnabled = ref(false)
const value = ref(0)

watch(
  [isEnabled, value],
  ([enabled, val]) => {
    // Only process when enabled
    if (enabled) {
      processValue(val)
    }
  }
)
```

### Watching with Validation

```typescript
import { ref, watch } from 'vue'

const email = ref('')
const emailError = ref('')
const isValidating = ref(false)

watch(email, async (newEmail) => {
  // Clear error on change
  emailError.value = ''
  
  if (!newEmail) return
  
  // Basic format check
  if (!newEmail.includes('@')) {
    emailError.value = 'Invalid email format'
    return
  }
  
  // Async validation (e.g., check availability)
  isValidating.value = true
  try {
    const response = await fetch(`/api/check-email?email=${newEmail}`)
    const { available } = await response.json()
    if (!available) {
      emailError.value = 'Email already registered'
    }
  } finally {
    isValidating.value = false
  }
}, { debounce: 500 })  // Note: debounce needs custom implementation
```

### Effect Scope for Cleanup

```typescript
import { ref, watch, effectScope } from 'vue'

function useFeature() {
  const scope = effectScope()
  
  scope.run(() => {
    const count = ref(0)
    
    watch(count, () => { /* ... */ })
    // More watchers, effects...
  })
  
  // Cleanup all effects at once
  function cleanup() {
    scope.stop()
  }
  
  return { cleanup }
}
```

## Resources

- [Watcher Callback Flush Timing](https://vuejs.org/guide/essentials/watchers.html#callback-flush-timing) — Understanding when watcher callbacks run

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*