---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-define-emits"
---

# Defining and Emitting Custom Events

Custom events allow child components to communicate with their parents. In Vue 3, we use `defineEmits()` to declare and type our events.

## Basic Event Emission

```vue
<script setup lang="ts">
const emit = defineEmits<{
  submit: []
  cancel: []
}>()

function handleSubmit() {
  // Perform submission logic...
  emit('submit')
}

function handleCancel() {
  emit('cancel')
}
</script>

<template>
  <div class="form-actions">
    <button @click="handleCancel">Cancel</button>
    <button @click="handleSubmit">Submit</button>
  </div>
</template>
```

## Events with Payloads

Pass data with your events:

```vue
<script setup lang="ts">
type User = {
  id: number
  name: string
  email: string
}

const emit = defineEmits<{
  select: [user: User]
  delete: [userId: number]
  search: [query: string, filters: string[]]
}>()

function handleUserSelect(user: User) {
  emit('select', user)
}

function handleDelete(userId: number) {
  if (confirm('Are you sure?')) {
    emit('delete', userId)
  }
}

function handleSearch(query: string) {
  emit('search', query, ['active', 'recent'])
}
</script>
```

## Runtime Declaration

Alternative syntax without TypeScript:

```vue
<script setup>
const emit = defineEmits({
  // No validation
  click: null,
  
  // With validation
  submit: (payload) => {
    if (payload.email && payload.password) {
      return true
    } else {
      console.warn('Invalid submit payload')
      return false
    }
  },
  
  // Simple validation
  increment: (amount) => typeof amount === 'number'
})
</script>
```

## Listening to Events

Parent components use `@event-name`:

```vue
<script setup lang="ts">
import UserCard from './UserCard.vue'

function handleSelect(user) {
  console.log('Selected:', user)
}

function handleDelete(userId) {
  console.log('Deleting:', userId)
}
</script>

<template>
  <UserCard
    @select="handleSelect"
    @delete="handleDelete"
  />
</template>
```

## Inline Handlers

For simple cases:

```vue
<template>
  <!-- Access payload with $event -->
  <SearchBox @search="searchQuery = $event" />
  
  <!-- Arrow function for multiple args -->
  <SearchBox @search="(query, filters) => performSearch(query, filters)" />
  
  <!-- Direct method reference (payload auto-passed) -->
  <UserCard @select="selectUser" />
</template>
```

## Event Naming Convention

Use camelCase in `defineEmits`, kebab-case in templates:

```vue
<script setup>
const emit = defineEmits({
  updateValue: null,      // camelCase in JS
  itemSelected: null,
  formSubmitted: null
})

emit('updateValue', newValue)
emit('itemSelected', item)
</script>
```

```vue
<!-- Parent template uses kebab-case -->
<MyComponent
  @update-value="handleUpdate"
  @item-selected="handleSelect"
  @form-submitted="handleSubmit"
/>
```

## Event Validation in Development

Validated events warn during development:

```vue
<script setup>
const emit = defineEmits({
  submit: (payload) => {
    if (!payload.email) {
      console.warn('submit event: missing email')
      return false
    }
    if (!payload.password || payload.password.length < 8) {
      console.warn('submit event: invalid password')
      return false
    }
    return true
  }
})

// This will warn in development
emit('submit', { email: 'test@test.com' })  // Missing password!
</script>
```

## Best Practices

1. **Always declare events** with `defineEmits`
2. **Use descriptive names** that describe what happened
3. **Validate payloads** for complex data
4. **Document expected payloads** with TypeScript types
5. **Keep payloads simple** - avoid deeply nested objects

## Resources

- [Component Events](https://vuejs.org/guide/components/events.html) — Official documentation on emitting custom events

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*