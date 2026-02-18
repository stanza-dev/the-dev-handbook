---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-watch-fundamentals"
---

# Understanding watch() and watchEffect()

Watchers let you perform side effects in response to reactive state changes. Vue provides two main APIs: `watch()` for explicit watching and `watchEffect()` for automatic dependency tracking.

## watch() Basics

`watch()` takes a source (what to watch) and a callback (what to do):

```typescript
import { ref, watch } from 'vue'

const count = ref(0)

// Watch a single ref
watch(count, (newValue, oldValue) => {
  console.log(`count changed: ${oldValue} → ${newValue}`)
})

// Trigger by changing the value
count.value = 1  // Logs: "count changed: 0 → 1"
```

## Watching Different Sources

### Single Ref

```typescript
const name = ref('Alice')

watch(name, (newName, oldName) => {
  console.log(`Name: ${oldName} → ${newName}`)
})
```

### Getter Function

Watch a computed expression or reactive property:

```typescript
import { reactive, watch } from 'vue'

const user = reactive({ name: 'Alice', age: 25 })

// Watch a specific property
watch(
  () => user.age,
  (newAge, oldAge) => {
    console.log(`Age: ${oldAge} → ${newAge}`)
  }
)

// Watch a computed expression
watch(
  () => user.age * 2,
  (doubled) => {
    console.log(`Doubled age: ${doubled}`)
  }
)
```

### Multiple Sources

```typescript
const firstName = ref('John')
const lastName = ref('Doe')

watch(
  [firstName, lastName],
  ([newFirst, newLast], [oldFirst, oldLast]) => {
    console.log(`Name changed from ${oldFirst} ${oldLast} to ${newFirst} ${newLast}`)
  }
)
```

## watchEffect()

`watchEffect()` automatically tracks all reactive dependencies:

```typescript
import { ref, watchEffect } from 'vue'

const count = ref(0)
const name = ref('Vue')

// Automatically tracks count and name
watchEffect(() => {
  console.log(`Count is ${count.value}, name is ${name.value}`)
})
// Runs immediately, then whenever count or name changes
```

## watch() vs watchEffect()

| Feature | watch() | watchEffect() |
|---------|---------|---------------|
| Explicit sources | ✅ Yes | ❌ Auto-tracked |
| Access to old value | ✅ Yes | ❌ No |
| Lazy by default | ✅ Yes | ❌ Runs immediately |
| Best for | Specific sources | Multiple deps |

### When to Use Each

```typescript
// Use watch() when:
// - You need the old value
watch(count, (newVal, oldVal) => {
  analytics.track('count_changed', { from: oldVal, to: newVal })
})

// - You want to watch specific sources
watch(
  () => route.params.id,
  (id) => fetchData(id)
)

// Use watchEffect() when:
// - You have multiple dependencies
watchEffect(() => {
  const url = `${baseUrl.value}/users/${userId.value}`
  // Both baseUrl and userId are tracked
})

// - You want immediate execution
watchEffect(() => {
  document.title = `${pageName.value} | My App`
})
```

## Callback Timing

By default, watchers run **after** component updates:

```typescript
import { ref, watch, watchEffect, watchPostEffect } from 'vue'

const count = ref(0)

// Default: runs after DOM updates
watch(count, () => {
  // DOM is updated here
  console.log(document.querySelector('#count')?.textContent)
})

// Same as above, more explicit
watch(count, callback, { flush: 'post' })

// Convenience alias
watchPostEffect(() => {
  // Runs after Vue updates the DOM
})
```

### Pre-flush (Before Update)

```typescript
watch(count, callback, { flush: 'pre' })
```

### Sync (Immediate, Before Everything)

```typescript
import { watchSyncEffect } from 'vue'

// Runs synchronously on every change
watchSyncEffect(() => {
  console.log('count is now', count.value)
})

// Or with watch
watch(count, callback, { flush: 'sync' })
```

**Warning**: Sync watchers run on EVERY change, even in batches. Use sparingly!

## Stopping Watchers

Watchers automatically stop when the component unmounts. For manual control:

```typescript
const stop = watch(source, callback)
const stopEffect = watchEffect(() => { /* ... */ })

// Later, manually stop
stop()
stopEffect()
```

## Cleanup

Run cleanup when the watcher re-runs or stops:

```typescript
watchEffect((onCleanup) => {
  const controller = new AbortController()
  
  fetch(url.value, { signal: controller.signal })
    .then(/* ... */)
  
  // Called before next run or when stopped
  onCleanup(() => {
    controller.abort()
  })
})

// With watch()
watch(id, async (newId, oldId, onCleanup) => {
  const controller = new AbortController()
  
  onCleanup(() => controller.abort())
  
  const data = await fetch(`/api/${newId}`, { 
    signal: controller.signal 
  })
})
```

## Resources

- [Watchers](https://vuejs.org/guide/essentials/watchers.html) — Official Vue documentation on watchers

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*