---
source_course: "vue-foundations"
source_lesson: "vue-foundations-next-steps"
---

# Next Steps in Your Vue Journey

Congratulations! You've completed the Vue Foundations course. You now have a solid understanding of:

- ✅ Vue's template syntax and directives
- ✅ Reactivity with `ref()`, `reactive()`, and `computed()`
- ✅ Event handling and form bindings
- ✅ Component creation with props and events
- ✅ Lifecycle hooks
- ✅ Project organization

## What to Learn Next

### 1. Vue Router

Add navigation to your single-page applications:

```bash
npm install vue-router@4
```

Key concepts:
- Route definitions and navigation
- Dynamic routes with parameters
- Navigation guards for authentication
- Nested routes for layouts

### 2. State Management with Pinia

Manage global state across your application:

```bash
npm install pinia
```

Key concepts:
- Creating stores
- State, getters, and actions
- Store composition
- DevTools integration

### 3. Advanced Component Patterns

Deep dive into:
- Slots for flexible component composition
- Provide/Inject for dependency injection
- Async components and Suspense
- Render functions and JSX

### 4. Composables

Create reusable logic:
- Custom composables
- VueUse library (100+ ready-made composables)
- Composable patterns and best practices

### 5. TypeScript Deep Dive

Leverage TypeScript fully:
- Generic components
- Typing complex props and emits
- Type utilities
- Type-safe routing

### 6. Testing

Ensure code quality:
- Unit testing with Vitest
- Component testing with Vue Test Utils
- End-to-end testing with Playwright/Cypress

### 7. Server-Side Rendering

Build SEO-friendly apps:
- Nuxt.js framework
- SSR vs SSG vs ISR
- Data fetching strategies

## Recommended Projects

Practice by building:

1. **Blog with Comments**
   - Markdown rendering
   - Comment system
   - Categories and tags

2. **E-commerce Store**
   - Product listings
   - Shopping cart (Pinia)
   - Checkout flow

3. **Dashboard Application**
   - Charts and data visualization
   - Real-time updates
   - User authentication

4. **Social Media Clone**
   - User profiles
   - Feed with infinite scroll
   - Real-time notifications

## Resources

### Official Resources
- [Vue.js Documentation](https://vuejs.org/guide/introduction.html)
- [Vue Router Documentation](https://router.vuejs.org/)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [VueUse](https://vueuse.org/)

### Community
- [Vue.js Discord](https://discord.com/invite/vue)
- [Vue Forum](https://forum.vuejs.org/)
- [Vue on Reddit](https://reddit.com/r/vuejs)

### Learning Platforms
- [Vue Mastery](https://www.vuemastery.com/)
- [Vue School](https://vueschool.io/)

## Quick Reference

```vue
<!-- Vue 3 Composition API Template -->
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

// Props
const props = defineProps<{
  title: string
  count?: number
}>()

// Emits
const emit = defineEmits<{
  update: [value: number]
}>()

// Reactive state
const message = ref('Hello')
const items = ref<string[]>([])

// Computed
const upperMessage = computed(() => message.value.toUpperCase())

// Watchers
watch(message, (newVal) => {
  console.log('Message changed:', newVal)
})

// Lifecycle
onMounted(() => {
  console.log('Component mounted')
})

// Methods
function handleClick() {
  emit('update', 42)
}
</script>

<template>
  <div>
    <h1>{{ title }}</h1>
    <p>{{ upperMessage }}</p>
    <button @click="handleClick">Click me</button>
    <ul>
      <li v-for="item in items" :key="item">{{ item }}</li>
    </ul>
  </div>
</template>

<style scoped>
/* Component styles */
</style>
```

Keep building, keep learning, and enjoy your Vue journey! 🎉

## Resources

- [Vue.js Documentation](https://vuejs.org/guide/introduction.html) — The complete official Vue.js documentation

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*