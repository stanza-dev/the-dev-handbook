---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-shallow-raw-utilities"
---

# shallowRef, shallowReactive, toRaw, and markRaw

These utilities give you fine-grained control over reactivity for performance optimization and integration with external systems.

## shallowRef()

Only the `.value` is reactive, not nested properties:

```typescript
import { shallowRef } from 'vue'

const state = shallowRef({
  nested: {
    count: 0
  }
})

// ❌ Does NOT trigger reactivity
state.value.nested.count++

// ✅ Triggers reactivity (replacing .value)
state.value = {
  nested: {
    count: 1
  }
}
```

### When to Use shallowRef

```typescript
import { shallowRef, triggerRef } from 'vue'

// Large data from external source
const largeData = shallowRef<BigDataset | null>(null)

async function loadData() {
  const data = await fetchHugeDataset()  // 10,000+ items
  largeData.value = data  // Single reactive update
}

// Manual trigger after mutation
function updateItem(index: number) {
  largeData.value!.items[index].processed = true
  triggerRef(largeData)  // Force update
}
```

## shallowReactive()

Only root-level properties are reactive:

```typescript
import { shallowReactive } from 'vue'

const state = shallowReactive({
  count: 0,          // Reactive
  nested: {          // The nested object itself is reactive
    value: 1         // But this is NOT reactive
  }
})

state.count++        // ✅ Triggers update
state.nested = {}    // ✅ Triggers update
state.nested.value++ // ❌ Does NOT trigger update
```

## toRaw()

Get the original object from a reactive proxy:

```typescript
import { reactive, toRaw } from 'vue'

const original = { count: 0 }
const proxy = reactive(original)

console.log(toRaw(proxy) === original)  // true

// Useful for:
// 1. Comparing with original
// 2. Passing to external libraries
// 3. Avoiding reactive overhead in performance-critical code
```

### Use Cases for toRaw

```typescript
import { reactive, toRaw } from 'vue'

const state = reactive({ items: [1, 2, 3] })

// External library that doesn't work with proxies
thirdPartyLibrary.process(toRaw(state.items))

// JSON serialization (proxies serialize fine, but toRaw is clearer)
const json = JSON.stringify(toRaw(state))

// Performance-critical loop
const raw = toRaw(state)
for (let i = 0; i < 1000000; i++) {
  // No proxy overhead
  raw.items[i % 3]
}
```

## markRaw()

Prevent an object from ever becoming reactive:

```typescript
import { markRaw, reactive } from 'vue'

const thirdPartyInstance = markRaw(new SomeHeavyClass())

const state = reactive({
  count: 0,
  external: thirdPartyInstance  // Will NOT be made reactive
})

// thirdPartyInstance stays as-is, no proxy wrapper
```

### When to Use markRaw

```typescript
import { markRaw, ref } from 'vue'

// 1. Class instances that shouldn't be reactive
class ExpensiveRenderer {
  canvas: HTMLCanvasElement
  // ... lots of internal state
}

const renderer = ref(markRaw(new ExpensiveRenderer()))

// 2. Large immutable data
const staticConfig = markRaw({
  // Thousands of static values
  countries: [/* ... */],
  currencies: [/* ... */]
})

// 3. Third-party objects
import { Chart } from 'chart.js'

const chart = ref<Chart | null>(null)

onMounted(() => {
  chart.value = markRaw(new Chart(/* ... */))
})
```

## Performance Comparison

```typescript
import { ref, shallowRef, reactive, shallowReactive } from 'vue'

// Deep reactivity: ~1000 proxies created
const deepData = reactive({
  level1: {
    level2: {
      // ... deeply nested
    }
  }
})

// Shallow: only 1 proxy
const shallowData = shallowReactive({
  level1: {
    // Not proxied
  }
})

// For large datasets, shallow + manual triggers
const bigData = shallowRef(generateLargeDataset())
```

## Decision Guide

| Scenario | Use |
|----------|-----|
| Normal reactive state | `ref()` / `reactive()` |
| Large datasets, external updates | `shallowRef()` + `triggerRef()` |
| Root-only reactivity needed | `shallowReactive()` |
| Passing to external libraries | `toRaw()` |
| Class instances, heavy objects | `markRaw()` |
| Performance-critical sections | `toRaw()` for reads |

## Resources

- [Reactivity Advanced](https://vuejs.org/api/reactivity-advanced.html) — Official documentation for advanced reactivity APIs

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*