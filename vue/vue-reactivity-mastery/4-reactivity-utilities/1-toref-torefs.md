---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-toref-torefs"
---

# toRef and toRefs for Destructuring

When working with reactive objects, destructuring loses reactivity. `toRef()` and `toRefs()` solve this problem.

## The Destructuring Problem

```typescript
import { reactive } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue'
})

// ❌ Loses reactivity!
let { count, name } = state
count++  // Doesn't update state.count
name = 'Vue 3'  // Doesn't update state.name
```

## toRef(): Single Property

`toRef()` creates a ref that syncs with a reactive object's property:

```typescript
import { reactive, toRef } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue'
})

// Create a ref that syncs with state.count
const countRef = toRef(state, 'count')

countRef.value++        // Also updates state.count
console.log(state.count)  // 1

state.count = 10        // Also updates countRef.value
console.log(countRef.value)  // 10
```

### toRef with Defaults (Vue 3.3+)

```typescript
import { reactive, toRef } from 'vue'

const state = reactive<{ name?: string }>({})

// Provide a default value
const name = toRef(state, 'name', 'Anonymous')
console.log(name.value)  // 'Anonymous'

state.name = 'Alice'
console.log(name.value)  // 'Alice'
```

## toRefs(): All Properties

`toRefs()` converts all properties to refs at once:

```typescript
import { reactive, toRefs } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue',
  active: true
})

// Convert all properties
const { count, name, active } = toRefs(state)

// All maintain two-way sync
count.value++
console.log(state.count)  // 1

name.value = 'Vue 3'
console.log(state.name)  // 'Vue 3'

state.active = false
console.log(active.value)  // false
```

## Common Use Cases

### Composable Return Values

```typescript
import { reactive, toRefs } from 'vue'

function useMouse() {
  const state = reactive({
    x: 0,
    y: 0
  })
  
  function update(event: MouseEvent) {
    state.x = event.clientX
    state.y = event.clientY
  }
  
  window.addEventListener('mousemove', update)
  
  // Return refs so consumers can destructure
  return toRefs(state)
}

// Consumer can destructure safely
const { x, y } = useMouse()
```

### Props Destructuring

```vue
<script setup lang="ts">
import { toRefs } from 'vue'

const props = defineProps<{
  count: number
  label: string
}>()

// ❌ Loses reactivity
// const { count, label } = props

// ✅ Maintains reactivity
const { count, label } = toRefs(props)

// Now you can use count.value and label.value
// and they'll update when props change
</script>
```

### Passing to Composables

```typescript
import { toRef } from 'vue'

function useFeature(value: Ref<number>) {
  // Works with ref
  watch(value, (newVal) => { /* ... */ })
}

const state = reactive({ count: 0 })

// Convert single property to ref
useFeature(toRef(state, 'count'))
```

## toRef vs computed

```typescript
import { reactive, toRef, computed } from 'vue'

const state = reactive({ count: 0 })

// toRef: two-way sync
const countRef = toRef(state, 'count')
countRef.value = 5  // Updates state.count

// computed: one-way (unless writable)
const countComputed = computed(() => state.count)
// countComputed.value = 5  // Error! Read-only
```

Use `toRef` when you need two-way binding. Use `computed` for derived values.

## Resources

- [toRef and toRefs](https://vuejs.org/api/reactivity-utilities.html#toref) — Official API documentation for toRef and toRefs

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*