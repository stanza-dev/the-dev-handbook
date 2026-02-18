---
source_course: "vue-foundations"
source_lesson: "vue-foundations-what-is-vue"
---

# What is Vue.js?

Vue (pronounced /vjuː/, like **view**) is a **progressive JavaScript framework** for building user interfaces. Created by Evan You in 2014, Vue has grown to become one of the most popular frontend frameworks in the world.

## The Progressive Framework

Vue is called "progressive" because it can scale between a simple library and a full-featured framework depending on your needs:

- **Enhance static HTML** without a build step
- **Embed as Web Components** on any page
- **Build Single-Page Applications** (SPA)
- **Create Fullstack applications** with Server-Side Rendering (SSR)
- **Generate Static Sites** (SSG) with Jamstack
- **Target multiple platforms**: desktop, mobile, WebGL, and even the terminal

You don't need to learn everything upfront. Vue grows with you as your project requirements evolve.

## Core Features

Vue's power comes from two foundational features:

### 1. Declarative Rendering

Vue extends standard HTML with a template syntax that lets you describe how the HTML should look based on JavaScript state:

```vue
<template>
  <div id="app">
    <h1>{{ message }}</h1>
    <p>Count: {{ count }}</p>
  </div>
</template>
```

The double curly braces `{{ }}` (called "mustaches") automatically display the value of your JavaScript variables.

### 2. Reactivity

Vue automatically tracks JavaScript state changes and efficiently updates the DOM when changes happen. You never manually update the page—Vue handles it for you:

```javascript
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++  // Vue automatically updates the DOM
}
```

## Single-File Components (SFC)

Vue uses a special file format called **Single-File Components** (`.vue` files) that combines HTML, JavaScript, and CSS in one file:

```vue
<script setup>
import { ref } from 'vue'

const greeting = ref('Hello Vue!')
</script>

<template>
  <p class="greeting">{{ greeting }}</p>
</template>

<style scoped>
.greeting {
  color: #42b883;
  font-weight: bold;
}
</style>
```

This structure keeps related code together, making components easy to understand and maintain.

## API Styles: Options vs Composition

Vue offers two ways to write component logic:

### Options API (Traditional)

Organizes code by **option type** (data, methods, computed, etc.):

```vue
<script>
export default {
  data() {
    return { count: 0 }
  },
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>
```

### Composition API (Modern - Recommended)

Organizes code by **logical concern** using imported functions:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
function increment() {
  count.value++
}
</script>
```

**This course uses the Composition API with `<script setup>`**, which is the recommended approach for Vue 3.5+. It offers:

- Less boilerplate code
- Better TypeScript support
- More flexible code organization
- Easier logic reuse with composables

## Why Choose Vue?

| Feature | Benefit |
|---------|----------|
| **Gentle Learning Curve** | Builds on HTML, CSS, and JavaScript you already know |
| **Excellent Documentation** | Among the best in the JavaScript ecosystem |
| **Versatile** | Works for projects of any size |
| **Performant** | Optimized virtual DOM and reactivity system |
| **Great Tooling** | Vue DevTools, Vite, and official libraries |
| **Active Ecosystem** | Vue Router, Pinia, Nuxt, and thousands of plugins |
| **Strong Community** | Millions of developers, regular updates |

## Resources

- [Vue.js Official Introduction](https://vuejs.org/guide/introduction.html) — The official Vue.js introduction covering core concepts

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*