---
source_course: "vue-typescript"
source_lesson: "vue-typescript-typing-emits-slots"
---

# Typing Emits and Slots

Type-safe events and slots ensure proper communication between components.

## Type-Based Emits

### Call Signature Syntax

```vue
<script setup lang="ts">
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
  (e: 'delete'): void  // No payload
}>()

// Usage
emit('change', 123)       // ✅
emit('update', 'hello')   // ✅
emit('delete')            // ✅
emit('change', 'wrong')   // ❌ Type error: string not assignable to number
emit('unknown')           // ❌ Type error: unknown event
</script>
```

### Named Tuple Syntax (Vue 3.3+)

```vue
<script setup lang="ts">
const emit = defineEmits<{
  change: [id: number]
  update: [value: string]
  submit: [data: FormData, silent?: boolean]
  delete: []  // No payload
}>()

// More concise and readable
emit('submit', formData, true)
</script>
```

### Complex Event Payloads

```vue
<script setup lang="ts">
interface User {
  id: number
  name: string
}

interface FilterOptions {
  search: string
  status: 'all' | 'active' | 'inactive'
  page: number
}

const emit = defineEmits<{
  select: [user: User]
  filter: [options: FilterOptions]
  sort: [field: keyof User, direction: 'asc' | 'desc']
}>()
</script>
```

## Runtime Emits with Validation

```vue
<script setup lang="ts">
const emit = defineEmits({
  // Validate submit payload
  submit: (payload: { email: string; password: string }) => {
    if (!payload.email.includes('@')) {
      console.warn('Invalid email')
      return false
    }
    return true
  },
  
  // No validation, just type
  cancel: null
})
</script>
```

## Typing Slots

### defineSlots (Vue 3.3+)

```vue
<script setup lang="ts">
const slots = defineSlots<{
  // Default slot with no props
  default(): any
  
  // Named slot with props
  header(props: { title: string }): any
  
  // Scoped slot
  item(props: { item: Item; index: number }): any
  
  // Optional slot
  footer?(): any
}>()

// Check if slot is provided
if (slots.footer) {
  // TypeScript knows footer exists here
}
</script>

<template>
  <div>
    <header>
      <slot name="header" title="Page Title" />
    </header>
    
    <main>
      <slot />  <!-- default -->
    </main>
    
    <ul>
      <li v-for="(item, index) in items" :key="item.id">
        <slot name="item" :item="item" :index="index" />
      </li>
    </ul>
    
    <footer v-if="slots.footer">
      <slot name="footer" />
    </footer>
  </div>
</template>
```

### Checking Slot Existence

```vue
<script setup lang="ts">
import { useSlots } from 'vue'

const slots = useSlots()

// Check if slots are provided
const hasHeader = computed(() => !!slots.header)
const hasFooter = computed(() => !!slots.footer)
</script>
```

## v-model Typing

### defineModel (Vue 3.4+)

```vue
<script setup lang="ts">
// Basic v-model
const modelValue = defineModel<string>()

// Required v-model
const modelValue = defineModel<string>({ required: true })

// With default
const modelValue = defineModel<string>({ default: '' })

// Named v-model
const firstName = defineModel<string>('firstName')
const lastName = defineModel<string>('lastName')

// With transform
const count = defineModel<number>({
  get(value) {
    return value ?? 0
  },
  set(value) {
    return Math.max(0, value)  // Ensure non-negative
  }
})
</script>
```

### Manual v-model Typing

```vue
<script setup lang="ts">
const props = defineProps<{
  modelValue: string
}>()

const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()

// Two-way binding helper
const value = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val)
})
</script>
```

## Complete Component Example

```vue
<script setup lang="ts">
import type { Component } from 'vue'

// Props
interface Props {
  title: string
  items: Item[]
  loading?: boolean
  icon?: Component
}

const props = withDefaults(defineProps<Props>(), {
  loading: false
})

// Emits
const emit = defineEmits<{
  select: [item: Item]
  delete: [id: number]
  'update:items': [items: Item[]]
}>()

// Slots
const slots = defineSlots<{
  default(): any
  header(props: { count: number }): any
  item(props: { item: Item; index: number }): any
  empty(): any
}>()

// v-model for items
const items = defineModel<Item[]>('items', { default: () => [] })
</script>
```

## Resources

- [Typing Component Emits](https://vuejs.org/guide/typescript/composition-api.html#typing-component-emits) — Official guide on typing emits

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*