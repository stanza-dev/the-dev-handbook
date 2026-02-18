---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-template-refs"
---

# Template Refs for DOM Access

Template refs provide direct access to DOM elements and child component instances, essential for DOM manipulation and imperative operations.

## Basic DOM Ref

```vue
<script setup>
import { ref, onMounted } from 'vue'

// Create a ref with same name as template ref
const inputRef = ref<HTMLInputElement | null>(null)

onMounted(() => {
  // Access the DOM element
  inputRef.value?.focus()
})

function selectAll() {
  inputRef.value?.select()
}
</script>

<template>
  <!-- Connect via ref attribute -->
  <input ref="inputRef" type="text" />
  <button @click="selectAll">Select All</button>
</template>
```

## Important: Refs are null until mounted

```vue
<script setup>
import { ref, onMounted, watchEffect } from 'vue'

const divRef = ref<HTMLDivElement | null>(null)

// ❌ null here - component not mounted yet
console.log(divRef.value)  // null

onMounted(() => {
  // ✅ Element exists now
  console.log(divRef.value)  // <div>...</div>
})

// Also works with watchEffect (runs after mount)
watchEffect(() => {
  if (divRef.value) {
    console.log('Div dimensions:', divRef.value.offsetWidth)
  }
})
</script>
```

## Multiple Refs with v-for

```vue
<script setup>
import { ref, onMounted } from 'vue'

const items = ref(['A', 'B', 'C'])

// Array of refs
const itemRefs = ref<HTMLLIElement[]>([])

onMounted(() => {
  console.log(itemRefs.value)  // [li, li, li]
  console.log(itemRefs.value.length)  // 3
})
</script>

<template>
  <ul>
    <li v-for="item in items" :key="item" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

**Note**: The array order is NOT guaranteed to match source array order.

## Function Refs

For more control, use a function:

```vue
<script setup>
import { ref } from 'vue'

const items = ref(['A', 'B', 'C'])
const itemRefs = ref<Map<string, HTMLElement>>(new Map())

function setItemRef(el: HTMLElement | null, key: string) {
  if (el) {
    itemRefs.value.set(key, el)
  } else {
    itemRefs.value.delete(key)  // Cleanup on unmount
  }
}
</script>

<template>
  <ul>
    <li 
      v-for="item in items" 
      :key="item"
      :ref="(el) => setItemRef(el, item)"
    >
      {{ item }}
    </li>
  </ul>
</template>
```

## Component Refs

```vue
<script setup>
import { ref, onMounted } from 'vue'
import ChildComponent from './ChildComponent.vue'

const childRef = ref<InstanceType<typeof ChildComponent> | null>(null)

onMounted(() => {
  // Access exposed properties/methods
  console.log(childRef.value?.count)
  childRef.value?.increment()
})
</script>

<template>
  <ChildComponent ref="childRef" />
</template>
```

### Child Component with defineExpose

```vue
<!-- ChildComponent.vue -->
<script setup>
import { ref } from 'vue'

const count = ref(0)
const privateData = ref('secret')

function increment() {
  count.value++
}

// Only exposed properties are accessible via ref
defineExpose({
  count,
  increment
})
</script>
```

## Common Use Cases

### Focus Management

```vue
<script setup>
import { ref, nextTick } from 'vue'

const inputRef = ref<HTMLInputElement | null>(null)
const showInput = ref(false)

async function openAndFocus() {
  showInput.value = true
  await nextTick()  // Wait for DOM update
  inputRef.value?.focus()
}
</script>

<template>
  <button @click="openAndFocus">Add Item</button>
  <input v-if="showInput" ref="inputRef" />
</template>
```

### Scroll Into View

```vue
<script setup>
import { ref } from 'vue'

const sectionRefs = ref<HTMLElement[]>([])

function scrollToSection(index: number) {
  sectionRefs.value[index]?.scrollIntoView({ 
    behavior: 'smooth',
    block: 'start'
  })
}
</script>
```

### Third-Party Library Integration

```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import Sortable from 'sortablejs'

const listRef = ref<HTMLElement | null>(null)
let sortable: Sortable | null = null

onMounted(() => {
  if (listRef.value) {
    sortable = new Sortable(listRef.value, {
      animation: 150,
      onEnd: handleSort
    })
  }
})

onUnmounted(() => {
  sortable?.destroy()
})
</script>

<template>
  <ul ref="listRef">
    <li v-for="item in items" :key="item.id">{{ item.name }}</li>
  </ul>
</template>
```

### Canvas/Video Access

```vue
<script setup>
import { ref, onMounted } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  const ctx = canvasRef.value?.getContext('2d')
  if (ctx) {
    ctx.fillStyle = 'blue'
    ctx.fillRect(0, 0, 100, 100)
  }
})
</script>

<template>
  <canvas ref="canvasRef" width="400" height="300" />
</template>
```

## Resources

- [Template Refs](https://vuejs.org/guide/essentials/template-refs.html) — Official documentation on template refs

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*