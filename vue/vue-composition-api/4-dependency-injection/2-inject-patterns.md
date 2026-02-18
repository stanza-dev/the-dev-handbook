---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-provide-inject-patterns"
---

# Advanced Provide/Inject Patterns

Let's explore sophisticated patterns for using provide/inject effectively.

## Plugin Pattern

Create a plugin that provides services:

```typescript
// toast-plugin.ts
import type { App, InjectionKey, Ref } from 'vue'
import { ref } from 'vue'

export type Toast = {
  id: number
  message: string
  type: 'success' | 'error' | 'info'
}

export type ToastService = {
  toasts: Ref<Toast[]>
  show: (message: string, type?: Toast['type']) => void
  dismiss: (id: number) => void
}

export const toastKey: InjectionKey<ToastService> = Symbol('toast')

export function createToastPlugin() {
  return {
    install(app: App) {
      const toasts = ref<Toast[]>([])
      let id = 0
      
      const service: ToastService = {
        toasts,
        show(message, type = 'info') {
          const toast = { id: ++id, message, type }
          toasts.value.push(toast)
          setTimeout(() => service.dismiss(toast.id), 3000)
        },
        dismiss(id) {
          const index = toasts.value.findIndex(t => t.id === id)
          if (index > -1) toasts.value.splice(index, 1)
        }
      }
      
      app.provide(toastKey, service)
    }
  }
}

// Composable for easy access
export function useToast() {
  const toast = inject(toastKey)
  if (!toast) throw new Error('Toast plugin not installed')
  return toast
}
```

```typescript
// main.ts
import { createApp } from 'vue'
import { createToastPlugin } from './toast-plugin'

const app = createApp(App)
app.use(createToastPlugin())
app.mount('#app')
```

```vue
<!-- Any component -->
<script setup>
import { useToast } from './toast-plugin'

const { show } = useToast()

function handleSave() {
  show('Saved successfully!', 'success')
}
</script>
```

## Context Pattern

Group related provisions:

```typescript
// auth-context.ts
import type { InjectionKey, Ref } from 'vue'
import { inject, provide, ref, computed } from 'vue'

type User = { id: number; name: string; email: string }

type AuthContext = {
  user: Ref<User | null>
  isLoggedIn: ComputedRef<boolean>
  login: (credentials: { email: string; password: string }) => Promise<void>
  logout: () => void
}

const authKey: InjectionKey<AuthContext> = Symbol('auth')

export function provideAuth() {
  const user = ref<User | null>(null)
  const isLoggedIn = computed(() => !!user.value)
  
  async function login(credentials: { email: string; password: string }) {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials)
    })
    user.value = await response.json()
  }
  
  function logout() {
    user.value = null
  }
  
  const context: AuthContext = {
    user,
    isLoggedIn,
    login,
    logout
  }
  
  provide(authKey, context)
  
  return context
}

export function useAuth() {
  const auth = inject(authKey)
  if (!auth) {
    throw new Error('useAuth must be used within an AuthProvider')
  }
  return auth
}
```

```vue
<!-- App.vue -->
<script setup>
import { provideAuth } from './auth-context'

provideAuth()  // Make auth available to all descendants
</script>

<!-- Any descendant -->
<script setup>
import { useAuth } from './auth-context'

const { user, isLoggedIn, logout } = useAuth()
</script>
```

## Form Context Pattern

```typescript
// form-context.ts
import type { InjectionKey } from 'vue'

type FormContext = {
  disabled: Ref<boolean>
  errors: Ref<Record<string, string>>
  registerField: (name: string) => void
  setFieldError: (name: string, error: string) => void
  clearFieldError: (name: string) => void
}

const formKey: InjectionKey<FormContext> = Symbol('form')

export function provideForm() {
  const disabled = ref(false)
  const errors = ref<Record<string, string>>({})
  const fields = ref<string[]>([])
  
  const context: FormContext = {
    disabled,
    errors,
    registerField(name) {
      fields.value.push(name)
    },
    setFieldError(name, error) {
      errors.value[name] = error
    },
    clearFieldError(name) {
      delete errors.value[name]
    }
  }
  
  provide(formKey, context)
  return context
}

export function useFormField(name: string) {
  const form = inject(formKey)
  if (!form) {
    throw new Error('useFormField must be used within a Form')
  }
  
  // Register this field
  form.registerField(name)
  
  const error = computed(() => form.errors.value[name])
  const hasError = computed(() => !!error.value)
  
  return {
    disabled: form.disabled,
    error,
    hasError,
    setError: (msg: string) => form.setFieldError(name, msg),
    clearError: () => form.clearFieldError(name)
  }
}
```

## Nested Providers (Overriding)

```vue
<!-- ThemeProvider.vue -->
<script setup>
import { provide } from 'vue'

const props = defineProps<{ theme: 'light' | 'dark' }>()

provide('theme', props.theme)
</script>

<template>
  <slot />
</template>

<!-- Usage: nested providers override -->
<template>
  <ThemeProvider theme="dark">
    <!-- Everything here uses dark theme -->
    <ThemeProvider theme="light">
      <!-- This section uses light theme -->
      <Card />  <!-- Injects 'light' -->
    </ThemeProvider>
  </ThemeProvider>
</template>
```

## Resources

- [Dependency Injection in Vue](https://vuejs.org/api/composition-api-dependency-injection.html) — API reference for provide/inject

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*