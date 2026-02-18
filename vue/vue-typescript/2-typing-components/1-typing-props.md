---
source_course: "vue-typescript"
source_lesson: "vue-typescript-typing-props"
---

# Typing Props

TypeScript props in Vue can be declared two ways: runtime declaration with type annotations, or pure type-based declaration.

## Type-Based Declaration (Recommended)

Pass types directly to `defineProps`:

```vue
<script setup lang="ts">
// Simple props
const props = defineProps<{
  title: string
  count: number
  disabled?: boolean  // Optional
}>()
</script>
```

### With Interface

```vue
<script setup lang="ts">
interface Props {
  title: string
  count: number
  disabled?: boolean
  items: string[]
  user: {
    id: number
    name: string
  }
}

const props = defineProps<Props>()
</script>
```

### With Type Alias

```vue
<script setup lang="ts">
type Status = 'pending' | 'active' | 'completed'

type Props = {
  id: number
  status: Status
  tags: string[]
}

const props = defineProps<Props>()
</script>
```

## Default Values with withDefaults

```vue
<script setup lang="ts">
interface Props {
  msg?: string
  count?: number
  items?: string[]
  user?: { name: string }
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'Hello',
  count: 0,
  // Use factory functions for non-primitive defaults
  items: () => [],
  user: () => ({ name: 'Guest' })
})
</script>
```

## Reactive Props Destructure (Vue 3.5+)

```vue
<script setup lang="ts">
interface Props {
  count: number
  msg?: string
}

// Destructure with defaults - stays reactive!
const { count, msg = 'Hello' } = defineProps<Props>()

// Can use directly
console.log(count, msg)
</script>
```

## Complex Prop Types

```vue
<script setup lang="ts">
import type { Component } from 'vue'

// Union types
type Size = 'small' | 'medium' | 'large'
type Variant = 'primary' | 'secondary' | 'danger'

// Generic types
type SelectOption<T> = {
  label: string
  value: T
  disabled?: boolean
}

interface Props {
  // Literal types
  size: Size
  variant: Variant
  
  // Array of objects
  options: SelectOption<string>[]
  
  // Function props
  formatter: (value: number) => string
  
  // Vue component as prop
  icon?: Component
  
  // Any HTML element
  element?: keyof HTMLElementTagNameMap
}

const props = defineProps<Props>()
</script>
```

## Runtime Declaration (Alternative)

Use when you need runtime validation:

```vue
<script setup lang="ts">
import type { PropType } from 'vue'

interface User {
  id: number
  name: string
}

const props = defineProps({
  // Simple types
  title: String,
  count: Number,
  
  // Required with type
  id: {
    type: Number,
    required: true
  },
  
  // Complex type with PropType
  user: {
    type: Object as PropType<User>,
    required: true
  },
  
  // Array of complex type
  items: {
    type: Array as PropType<User[]>,
    default: () => []
  },
  
  // Function prop
  callback: {
    type: Function as PropType<(id: number) => void>,
    required: false
  },
  
  // Union type with validator
  status: {
    type: String as PropType<'active' | 'inactive'>,
    default: 'active',
    validator: (value: string) => ['active', 'inactive'].includes(value)
  }
})
</script>
```

## Importing Types for Props

```typescript
// types/user.ts
export interface User {
  id: number
  name: string
  email: string
}

export type UserRole = 'admin' | 'user' | 'guest'
```

```vue
<script setup lang="ts">
import type { User, UserRole } from '@/types/user'

const props = defineProps<{
  user: User
  role: UserRole
}>()
</script>
```

## Resources

- [Typing Component Props](https://vuejs.org/guide/typescript/composition-api.html#typing-component-props) — Official guide on typing props

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*