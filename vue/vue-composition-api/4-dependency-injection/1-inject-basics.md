---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-provide-inject-basics"
---

# Provide/Inject Fundamentals

Provide/Inject allows ancestor components to serve as dependency providers for all descendants, avoiding "prop drilling" through intermediate components.

## The Problem: Prop Drilling

```
Grandparent (has theme)
    └─ Parent (just passes theme)
        └─ Child (just passes theme)
            └─ GrandChild (uses theme)
```

Every intermediate component must declare and pass the prop!

## The Solution: Provide/Inject

```
Grandparent (provides theme)
    └─ Parent (unaware of theme)
        └─ Child (unaware of theme)
            └─ GrandChild (injects theme)
```

## Basic Usage

### Provider Component

```vue
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
const user = ref({ name: 'Alice', role: 'admin' })

// Provide values
provide('theme', theme)
provide('user', user)

// Provide methods too
function toggleTheme() {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
}
provide('toggleTheme', toggleTheme)
</script>
```

### Consumer Component

```vue
<script setup>
import { inject } from 'vue'

// Inject values
const theme = inject('theme')
const user = inject('user')
const toggleTheme = inject('toggleTheme')
</script>

<template>
  <div :class="theme">
    <p>Welcome, {{ user.name }}</p>
    <button @click="toggleTheme">Toggle Theme</button>
  </div>
</template>
```

## Default Values

```vue
<script setup>
import { inject } from 'vue'

// With default value
const theme = inject('theme', 'light')

// With factory function (for expensive defaults)
const heavyDefault = inject('heavy', () => createExpensiveObject(), true)
</script>
```

## Providing at App Level

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// App-wide provides
app.provide('appName', 'My App')
app.provide('apiBase', 'https://api.example.com')

app.mount('#app')
```

## Reactivity with Provide/Inject

Provided refs stay reactive:

```vue
<!-- Provider -->
<script setup>
import { ref, provide } from 'vue'

const count = ref(0)
provide('count', count)  // Reactive!

setInterval(() => count.value++, 1000)
</script>

<!-- Consumer -->
<script setup>
import { inject } from 'vue'

const count = inject('count')  // Updates automatically!
</script>

<template>
  <p>Count: {{ count }}</p>  <!-- Updates every second -->
</template>
```

## Read-Only Provisions

Prevent consumers from mutating:

```vue
<script setup>
import { ref, provide, readonly } from 'vue'

const count = ref(0)

// Provide read-only version
provide('count', readonly(count))

// Provide update method separately
provide('incrementCount', () => count.value++)
</script>
```

## Working with TypeScript

### Using Injection Keys

```typescript
// keys.ts
import type { InjectionKey, Ref } from 'vue'

export type Theme = 'light' | 'dark'

export const themeKey: InjectionKey<Ref<Theme>> = Symbol('theme')
export const toggleThemeKey: InjectionKey<() => void> = Symbol('toggleTheme')
```

```vue
<!-- Provider -->
<script setup lang="ts">
import { ref, provide } from 'vue'
import { themeKey, toggleThemeKey, type Theme } from './keys'

const theme = ref<Theme>('dark')

provide(themeKey, theme)
provide(toggleThemeKey, () => {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
})
</script>

<!-- Consumer -->
<script setup lang="ts">
import { inject } from 'vue'
import { themeKey, toggleThemeKey } from './keys'

const theme = inject(themeKey)!  // Ref<Theme>
const toggleTheme = inject(toggleThemeKey)!  // () => void
</script>
```

## When to Use Provide/Inject

✅ Good use cases:
- Theme/locale across app
- Auth/user state
- Shared configuration
- Plugin data
- Component library internals

❌ Not ideal for:
- Simple parent-child communication (use props)
- Global state management (use Pinia)
- Sibling communication (use events or state)

## Resources

- [Provide/Inject](https://vuejs.org/guide/components/provide-inject.html) — Official documentation on provide/inject

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*