---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-provide-inject-basics"
---

# Dependency Injection with Provide/Inject

Provide/Inject is Vue's dependency injection system. It allows ancestor components to provide data that any descendant can inject, regardless of nesting depth.

## The Problem: Prop Drilling

Without provide/inject, passing data through many levels requires "prop drilling":

```
Grandparent → Parent → Child → GrandChild
   (prop)      (prop)   (prop)   (uses it)
```

Every intermediate component must accept and pass the prop, even if it doesn't use it.

## Basic Provide/Inject

### Provider Component

```vue
<!-- App.vue or ancestor component -->
<script setup lang="ts">
import { provide } from 'vue'

// Provide a static value
provide('appName', 'My Application')
provide('version', '1.0.0')
provide('apiUrl', 'https://api.example.com')
</script>
```

### Injector Component

```vue
<!-- Any descendant component -->
<script setup lang="ts">
import { inject } from 'vue'

const appName = inject('appName')
const version = inject('version')
const apiUrl = inject('apiUrl')
</script>

<template>
  <footer>
    {{ appName }} v{{ version }}
  </footer>
</template>
```

## Providing Reactive State

Provide reactive values that update in all injectors:

```vue
<!-- ThemeProvider.vue -->
<script setup lang="ts">
import { ref, provide, readonly } from 'vue'

const theme = ref<'light' | 'dark'>('light')
const primaryColor = ref('#42b883')

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

function setPrimaryColor(color: string) {
  primaryColor.value = color
}

// Provide reactive values and methods
provide('theme', {
  current: readonly(theme),  // Readonly to prevent direct mutation
  primaryColor: readonly(primaryColor),
  toggle: toggleTheme,
  setPrimaryColor
})
</script>

<template>
  <div :class="['app', theme]">
    <slot></slot>
  </div>
</template>
```

```vue
<!-- Any descendant -->
<script setup lang="ts">
import { inject } from 'vue'

const theme = inject('theme')
</script>

<template>
  <div :style="{ color: theme.primaryColor }">
    Current theme: {{ theme.current }}
    <button @click="theme.toggle">Toggle Theme</button>
  </div>
</template>
```

## Default Values

Provide fallbacks when injection key isn't found:

```vue
<script setup lang="ts">
import { inject } from 'vue'

// With default value
const theme = inject('theme', 'light')
const config = inject('config', { debug: false })

// With factory function (for objects)
const user = inject('user', () => ({ name: 'Guest' }))
</script>
```

## Symbol Keys for Type Safety

Use Symbols to avoid naming collisions:

```typescript
// src/injection-keys.ts
import type { InjectionKey, Ref } from 'vue'

export type Theme = 'light' | 'dark'

export type ThemeContext = {
  current: Readonly<Ref<Theme>>
  toggle: () => void
}

export const ThemeKey: InjectionKey<ThemeContext> = Symbol('theme')
export const UserKey: InjectionKey<User | null> = Symbol('user')
```

```vue
<!-- Provider -->
<script setup lang="ts">
import { provide, ref, readonly } from 'vue'
import { ThemeKey } from '@/injection-keys'

const theme = ref<'light' | 'dark'>('light')

provide(ThemeKey, {
  current: readonly(theme),
  toggle: () => { theme.value = theme.value === 'light' ? 'dark' : 'light' }
})
</script>
```

```vue
<!-- Consumer -->
<script setup lang="ts">
import { inject } from 'vue'
import { ThemeKey } from '@/injection-keys'

// TypeScript knows the exact type!
const theme = inject(ThemeKey)

if (theme) {
  console.log(theme.current.value)  // 'light' | 'dark'
}
</script>
```

## App-Level Provide

Provide values available to the entire application:

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// Available in all components
app.provide('globalConfig', {
  apiUrl: import.meta.env.VITE_API_URL,
  appVersion: '1.0.0'
})

app.mount('#app')
```

## Best Practices

1. **Use readonly** for provided refs to prevent uncontrolled mutations
2. **Provide update functions** instead of mutable refs
3. **Use Symbol keys** for type safety and avoiding collisions
4. **Provide at the right level** - not everything needs to be app-level
5. **Document what you provide** for team clarity

## Resources

- [Provide/Inject](https://vuejs.org/guide/components/provide-inject.html) — Official Vue documentation on provide/inject

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*