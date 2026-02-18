---
source_course: "vue-foundations"
source_lesson: "vue-foundations-project-organization"
---

# Project Organization Best Practices

As your Vue application grows, good organization becomes essential. Let's establish patterns that scale from small projects to large applications.

## Recommended Folder Structure

```
src/
├── assets/              # Static assets (images, fonts)
│   ├── images/
│   └── styles/
│       ├── main.css
│       └── variables.css
│
├── components/          # Reusable components
│   ├── common/          # Generic, app-wide components
│   │   ├── BaseButton.vue
│   │   ├── BaseInput.vue
│   │   ├── BaseModal.vue
│   │   └── BaseCard.vue
│   ├── layout/          # Layout components
│   │   ├── AppHeader.vue
│   │   ├── AppFooter.vue
│   │   └── AppSidebar.vue
│   └── [feature]/       # Feature-specific components
│       ├── TodoItem.vue
│       └── TodoList.vue
│
├── composables/         # Reusable composition functions
│   ├── useAuth.ts
│   ├── useFetch.ts
│   └── useLocalStorage.ts
│
├── views/               # Page-level components
│   ├── HomeView.vue
│   ├── AboutView.vue
│   └── NotFoundView.vue
│
├── types/               # TypeScript type definitions
│   └── index.ts
│
├── utils/               # Utility functions
│   ├── formatters.ts
│   └── validators.ts
│
├── App.vue              # Root component
└── main.ts              # Application entry point
```

## Component Naming Patterns

### Base Components

Prefix generic components with `Base`, `App`, or `V`:

```
BaseButton.vue      # Generic button
BaseInput.vue       # Generic input
BaseCard.vue        # Generic card
AppHeader.vue       # App-specific header
```

### Single-Instance Components

Prefix components used only once with `The`:

```
TheNavbar.vue       # Only one navbar
TheSidebar.vue      # Only one sidebar
TheFooter.vue       # Only one footer
```

### Tightly Coupled Components

Name child components with parent as prefix:

```
TodoList.vue
TodoListItem.vue
TodoListItemButton.vue

SearchForm.vue
SearchFormInput.vue
SearchFormButton.vue
```

## File Organization Tips

### Keep Related Code Together

```
components/
└── todo/
    ├── TodoList.vue
    ├── TodoItem.vue
    ├── TodoForm.vue
    └── types.ts          # Types specific to this feature
```

### Co-locate Tests

```
components/
└── todo/
    ├── TodoList.vue
    ├── TodoList.spec.ts  # Test next to component
    └── ...
```

## Creating Type Definitions

```typescript
// src/types/index.ts
export type User = {
  id: number
  name: string
  email: string
  avatar?: string
}

export type Todo = {
  id: number
  text: string
  done: boolean
  createdAt: Date
  userId: number
}

export type ApiResponse<T> = {
  data: T
  message: string
  success: boolean
}
```

## Creating Utility Functions

```typescript
// src/utils/formatters.ts
export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  }).format(date)
}

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(amount)
}

export function truncate(text: string, length: number): string {
  if (text.length <= length) return text
  return text.slice(0, length) + '...'
}
```

## Using Utilities in Components

```vue
<script setup lang="ts">
import { formatDate, formatCurrency } from '@/utils/formatters'
import type { Product } from '@/types'

const props = defineProps<{
  product: Product
}>()
</script>

<template>
  <div class="product">
    <h3>{{ product.name }}</h3>
    <p>{{ formatCurrency(product.price) }}</p>
    <small>Added {{ formatDate(product.createdAt) }}</small>
  </div>
</template>
```

## Path Aliases

Configure path aliases in `vite.config.ts` for cleaner imports:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { fileURLToPath, URL } from 'node:url'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

Now you can import like this:

```typescript
// Instead of: import Button from '../../../components/common/BaseButton.vue'
import BaseButton from '@/components/common/BaseButton.vue'
import { formatDate } from '@/utils/formatters'
import type { User } from '@/types'
```

## Resources

- [Vue Style Guide](https://vuejs.org/style-guide/) — Official Vue.js style guide for best practices

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*