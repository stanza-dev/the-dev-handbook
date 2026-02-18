---
source_course: "vue-typescript"
source_lesson: "vue-typescript-typing-watchers"
---

# Typing Watch and WatchEffect

Watchers are generally well-inferred, but understanding the types helps with complex scenarios.

## watch() Typing

### Basic Watch

```typescript
import { ref, watch } from 'vue'

const count = ref(0)

// Types are inferred from source
watch(count, (newValue, oldValue) => {
  // newValue: number
  // oldValue: number
  console.log(`Changed from ${oldValue} to ${newValue}`)
})
```

### Watching Getter

```typescript
import { reactive, watch } from 'vue'

const user = reactive({ name: 'Alice', age: 25 })

// Watch specific property
watch(
  () => user.age,
  (newAge, oldAge) => {
    // newAge: number
    // oldAge: number
  }
)

// Watch computed expression
watch(
  () => user.age >= 18,
  (isAdult) => {
    // isAdult: boolean
  }
)
```

### Watching Multiple Sources

```typescript
import { ref, watch } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')
const age = ref(30)

watch(
  [firstName, lastName, age],
  ([newFirst, newLast, newAge], [oldFirst, oldLast, oldAge]) => {
    // newFirst: string, oldFirst: string
    // newLast: string, oldLast: string
    // newAge: number, oldAge: number
  }
)

// Mixed sources
watch(
  [firstName, () => age.value * 2],
  ([name, doubleAge]) => {
    // name: string
    // doubleAge: number
  }
)
```

### Watch Options Typing

```typescript
import { ref, watch, type WatchOptions } from 'vue'

const data = ref<Data | null>(null)

const options: WatchOptions = {
  immediate: true,
  deep: true,
  flush: 'post',
  onTrack(e) {
    console.log('tracked', e)
  },
  onTrigger(e) {
    console.log('triggered', e)
  }
}

watch(data, (newData) => {
  // newData could be null on first run with immediate
  if (newData) {
    process(newData)
  }
}, options)
```

## watchEffect() Typing

```typescript
import { ref, watchEffect, type WatchEffectReturn } from 'vue'

const count = ref(0)

// Stop handle is returned
const stop: WatchEffectReturn = watchEffect(() => {
  console.log('Count:', count.value)
})

// With cleanup
watchEffect((onCleanup) => {
  const controller = new AbortController()
  
  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .then(data => {
      // Process data
    })
  
  onCleanup(() => {
    controller.abort()
  })
})
```

## Async Watch Handlers

```typescript
import { ref, watch } from 'vue'

const userId = ref<number>(1)
const user = ref<User | null>(null)
const error = ref<Error | null>(null)

// Async handler
watch(userId, async (newId) => {
  error.value = null
  try {
    user.value = await fetchUser(newId)
  } catch (e) {
    error.value = e as Error
    user.value = null
  }
})
```

## Watch with Cleanup and Abort

```typescript
import { ref, watch } from 'vue'

const searchQuery = ref('')
const results = ref<SearchResult[]>([])

watch(searchQuery, async (query, oldQuery, onCleanup) => {
  const controller = new AbortController()
  
  onCleanup(() => {
    controller.abort()
  })
  
  if (!query) {
    results.value = []
    return
  }
  
  try {
    const response = await fetch(
      `/api/search?q=${query}`,
      { signal: controller.signal }
    )
    results.value = await response.json()
  } catch (e) {
    if ((e as Error).name !== 'AbortError') {
      console.error(e)
    }
  }
})
```

## Type Guards in Watchers

```typescript
import { ref, watch } from 'vue'

type Status = 'idle' | 'loading' | 'success' | 'error'

const status = ref<Status>('idle')
const data = ref<Data | null>(null)
const error = ref<Error | null>(null)

watch(status, (newStatus) => {
  switch (newStatus) {
    case 'success':
      // data should be available
      if (data.value) {
        processData(data.value)
      }
      break
    case 'error':
      // error should be available
      if (error.value) {
        logError(error.value)
      }
      break
  }
})
```

## Resources

- [Watchers](https://vuejs.org/guide/essentials/watchers.html) — Official guide on watchers

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*