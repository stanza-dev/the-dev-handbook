---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-script-setup-fundamentals"
---

# Understanding script setup

`<script setup>` is the recommended syntax for using the Composition API in Single-File Components. It provides a cleaner, more concise way to write components.

## Why script setup?

### Before (Options API / setup function)

```vue
<script>
import { ref, computed } from 'vue'
import MyComponent from './MyComponent.vue'

export default {
  components: { MyComponent },
  setup() {
    const count = ref(0)
    const double = computed(() => count.value * 2)
    
    function increment() {
      count.value++
    }
    
    // Must explicitly return everything
    return {
      count,
      double,
      increment
    }
  }
}
</script>
```

### After (script setup)

```vue
<script setup>
import { ref, computed } from 'vue'
import MyComponent from './MyComponent.vue'  // Auto-registered!

const count = ref(0)
const double = computed(() => count.value * 2)

function increment() {
  count.value++
}
// Everything is automatically available in template!
</script>
```

## Key Benefits

1. **Less boilerplate**: No `return` statement, no `components` option
2. **Better TypeScript**: Types flow naturally without extra annotations
3. **Better runtime performance**: Template is compiled in the same scope
4. **Auto-imports**: Components and directives are automatically registered

## What's Available in Template

Everything at the top level is automatically exposed:

```vue
<script setup>
import { ref, computed } from 'vue'
import BaseButton from './BaseButton.vue'  // Component
import vFocus from './directives/focus'    // Directive

// Reactive state
const count = ref(0)
const user = ref({ name: 'Alice' })

// Computed
const double = computed(() => count.value * 2)

// Functions
function increment() { count.value++ }
const greet = () => alert('Hello!')

// Constants
const MAX_COUNT = 100
</script>

<template>
  <!-- All of these work -->
  <BaseButton @click="increment">Count: {{ count }}</BaseButton>
  <p>Double: {{ double }}</p>
  <p>User: {{ user.name }}</p>
  <p>Max: {{ MAX_COUNT }}</p>
  <input v-focus />
</template>
```

## Using Alongside Regular script

You can have both `<script>` and `<script setup>` in the same component:

```vue
<script>
// Regular script for:
// - Named exports
// - Options that can't be expressed in setup
// - Code that should only run once (side effects)

export const componentName = 'MyCounter'

export default {
  name: 'MyCounter',
  inheritAttrs: false,
  customOptions: {
    // Plugin-specific options
  }
}
</script>

<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>
```

## Top-Level await

`<script setup>` supports top-level `await`:

```vue
<script setup>
const user = await fetchUser()  // Pauses component setup
const posts = await fetchUserPosts(user.id)
</script>
```

**Note**: This makes the component an async component, requiring a parent `<Suspense>`:

```vue
<template>
  <Suspense>
    <AsyncComponent />
    <template #fallback>Loading...</template>
  </Suspense>
</template>
```

## Restrictions

A few things don't work in `<script setup>`:

- No `this` access (it's `undefined`)
- No named exports from script setup itself
- Can't use `src` attribute with script setup

## Resources

- [script setup Documentation](https://vuejs.org/api/sfc-script-setup.html) — Complete guide to script setup

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*