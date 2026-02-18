---
source_course: "vue-foundations"
source_lesson: "vue-foundations-understanding-sfc"
---

# Understanding Single-File Components

Single-File Components (SFCs) are Vue's signature feature. They let you define a component's logic, template, and styles in a single `.vue` file, keeping related code together.

## Anatomy of an SFC

Every `.vue` file can contain three types of top-level blocks:

```vue
<script setup lang="ts">
// JavaScript/TypeScript logic
</script>

<template>
  <!-- HTML template -->
</template>

<style scoped>
/* CSS styles */
</style>
```

### The `<script setup>` Block

Contains your component's JavaScript or TypeScript logic:

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import MyComponent from './MyComponent.vue'

// Reactive state
const count = ref(0)
const name = ref('Vue')

// Computed properties
const doubleCount = computed(() => count.value * 2)

// Functions
function increment() {
  count.value++
}
</script>
```

The `setup` attribute is a compile-time hint that enables:
- **Automatic exposure**: All top-level bindings are available in the template
- **Less boilerplate**: No need for `return` statements or `export default`
- **Better performance**: More efficient compiled output

### The `<template>` Block

Contains your HTML structure with Vue's template syntax:

```vue
<template>
  <div class="container">
    <h1>{{ name }}</h1>
    <p>Count: {{ count }}</p>
    <p>Double: {{ doubleCount }}</p>
    <button @click="increment">+1</button>
    <MyComponent />
  </div>
</template>
```

Key features:
- **Interpolation**: `{{ expression }}` for displaying values
- **Directives**: `v-if`, `v-for`, `@click`, `:prop` for dynamic behavior
- **Component usage**: Import and use other components directly

### The `<style>` Block

Contains your CSS styles:

```vue
<style scoped>
.container {
  padding: 1rem;
}

button {
  background: #42b883;
  color: white;
}
</style>
```

The `scoped` attribute is **crucial**—it ensures styles only apply to this component, preventing CSS conflicts.

## Scoped Styles Deep Dive

Without `scoped`, styles are global and can affect other components:

```vue
<!-- Global styles - affects ALL buttons in your app -->
<style>
button {
  background: red;
}
</style>
```

With `scoped`, Vue adds unique attributes to elements:

```vue
<!-- Scoped styles - only affects THIS component -->
<style scoped>
button {
  background: green;
}
</style>

<!-- Compiled output:
<button data-v-7ba5bd90>Click</button>

button[data-v-7ba5bd90] {
  background: green;
}
-->
```

## CSS Features in SFCs

### CSS Modules

Generate unique class names automatically:

```vue
<template>
  <p :class="$style.red">Red text</p>
</template>

<style module>
.red {
  color: red;
}
</style>
```

### v-bind() in CSS

Use JavaScript values in your styles:

```vue
<script setup>
import { ref } from 'vue'

const color = ref('red')
const fontSize = ref(16)
</script>

<template>
  <p class="text">Dynamic styling!</p>
  <button @click="color = 'blue'">Change to blue</button>
</template>

<style scoped>
.text {
  color: v-bind(color);
  font-size: v-bind(fontSize + 'px');
}
</style>
```

### Multiple Style Blocks

You can have both scoped and global styles:

```vue
<style>
/* Global styles */
body {
  margin: 0;
}
</style>

<style scoped>
/* Component styles */
.wrapper {
  padding: 1rem;
}
</style>
```

## Pre-processors

Use SCSS, Less, or other pre-processors:

```vue
<style lang="scss" scoped>
$primary: #42b883;

.button {
  background: $primary;
  
  &:hover {
    background: darken($primary, 10%);
  }
}
</style>
```

Install the pre-processor:

```bash
npm install -D sass
```

## Best Practices

1. **Always use `scoped`** unless you specifically need global styles
2. **Keep components focused** - if your SFC is getting large, split it into smaller components
3. **Use TypeScript** with `lang="ts"` for better type checking
4. **Order blocks consistently**: `<script>`, `<template>`, `<style>`

## Resources

- [Single-File Components](https://vuejs.org/guide/scaling-up/sfc.html) — Official documentation on Vue Single-File Components

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*