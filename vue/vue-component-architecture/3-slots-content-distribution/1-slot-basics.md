---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-slot-basics"
---

# Understanding Slots

Slots allow parent components to pass template content into child components. They're essential for creating flexible, reusable components.

## The Default Slot

The simplest slot usage:

```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <slot></slot>
  </div>
</template>
```

```vue
<!-- Parent -->
<Card>
  <h2>Card Title</h2>
  <p>This content goes into the slot!</p>
</Card>
```

The content between `<Card>` tags replaces `<slot>`.

## Fallback Content

Provide default content when the slot is empty:

```vue
<!-- SubmitButton.vue -->
<template>
  <button type="submit" class="btn">
    <slot>Submit</slot>
  </button>
</template>
```

```vue
<!-- Parent -->
<SubmitButton />               <!-- Shows: Submit -->
<SubmitButton>Save Changes</SubmitButton>  <!-- Shows: Save Changes -->
<SubmitButton>                 <!-- Shows: Submit (empty content) -->
</SubmitButton>
```

## Named Slots

Create multiple slots with names:

```vue
<!-- BaseLayout.vue -->
<template>
  <div class="layout">
    <header>
      <slot name="header"></slot>
    </header>
    
    <main>
      <slot></slot>  <!-- Default slot -->
    </main>
    
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

Use `v-slot` directive (or `#` shorthand) to target named slots:

```vue
<!-- Parent -->
<BaseLayout>
  <template #header>
    <h1>My Page Title</h1>
    <nav>Navigation here</nav>
  </template>
  
  <!-- Default slot content -->
  <p>Main content goes here...</p>
  <p>More content...</p>
  
  <template #footer>
    <p>Copyright 2024</p>
  </template>
</BaseLayout>
```

## Slot Shorthand: #

Use `#` instead of `v-slot:`:

```vue
<template>
  <!-- Full syntax -->
  <template v-slot:header>...</template>
  <template v-slot:footer>...</template>
  
  <!-- Shorthand -->
  <template #header>...</template>
  <template #footer>...</template>
</template>
```

## Conditional Slots

Check if a slot has content with `$slots`:

```vue
<!-- Card.vue -->
<script setup>
import { useSlots } from 'vue'

const slots = useSlots()
</script>

<template>
  <div class="card">
    <header v-if="slots.header" class="card-header">
      <slot name="header"></slot>
    </header>
    
    <div class="card-body">
      <slot></slot>
    </div>
    
    <footer v-if="slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

## Dynamic Slot Names

Use dynamic values for slot names:

```vue
<script setup>
import { ref } from 'vue'

const currentSlot = ref('header')
</script>

<template>
  <BaseLayout>
    <template #[currentSlot]>
      <p>This content goes to: {{ currentSlot }}</p>
    </template>
  </BaseLayout>
  
  <button @click="currentSlot = 'footer'">Move to Footer</button>
</template>
```

## Practical Example: Modal Component

```vue
<!-- Modal.vue -->
<script setup lang="ts">
const props = defineProps<{
  isOpen: boolean
}>()

const emit = defineEmits<{
  close: []
}>()
</script>

<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click.self="emit('close')">
      <div class="modal">
        <header v-if="$slots.header" class="modal-header">
          <slot name="header"></slot>
          <button class="close-btn" @click="emit('close')">×</button>
        </header>
        
        <div class="modal-body">
          <slot></slot>
        </div>
        
        <footer v-if="$slots.footer" class="modal-footer">
          <slot name="footer"></slot>
        </footer>
      </div>
    </div>
  </Teleport>
</template>
```

```vue
<!-- Parent -->
<Modal :is-open="showModal" @close="showModal = false">
  <template #header>
    <h2>Confirm Action</h2>
  </template>
  
  <p>Are you sure you want to proceed?</p>
  
  <template #footer>
    <button @click="showModal = false">Cancel</button>
    <button @click="confirm">Confirm</button>
  </template>
</Modal>
```

## Resources

- [Slots Documentation](https://vuejs.org/guide/components/slots.html) — Official Vue documentation on slots

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*