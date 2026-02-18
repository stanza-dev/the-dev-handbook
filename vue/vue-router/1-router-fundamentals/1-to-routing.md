---
source_course: "vue-router"
source_lesson: "vue-router-introduction-to-routing"
---

# Introduction to Client-Side Routing

Vue Router is the official routing library for Vue.js, enabling you to build single-page applications (SPAs) with multiple views.

## What is Client-Side Routing?

In traditional websites, clicking a link causes a full page reload:

```
Browser → Request /about → Server → Full HTML page → Render
```

With client-side routing:

```
Browser → Change URL → JavaScript handles it → Update view → No server round-trip
```

## Benefits of SPAs with Vue Router

1. **Faster navigation**: No full page reloads
2. **Smoother UX**: Transitions between views
3. **State preservation**: Keep data across navigations
4. **Native-like feel**: Instant responses

## Installation

```bash
npm install vue-router@4
# or
pnpm add vue-router@4
```

## Basic Setup

### 1. Create Router Configuration

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/views/Home.vue'
import About from '@/views/About.vue'

const routes = [
  {
    path: '/',
    name: 'home',
    component: Home
  },
  {
    path: '/about',
    name: 'about',
    component: About
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### 2. Register Router in App

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(router)
app.mount('#app')
```

### 3. Add Router View and Links

```vue
<!-- App.vue -->
<template>
  <nav>
    <router-link to="/">Home</router-link>
    <router-link to="/about">About</router-link>
  </nav>
  
  <!-- Route components render here -->
  <router-view />
</template>
```

## Core Concepts

### router-link

Declarative navigation component:

```vue
<!-- Simple link -->
<router-link to="/about">About</router-link>

<!-- Named route -->
<router-link :to="{ name: 'about' }">About</router-link>

<!-- With params -->
<router-link :to="{ name: 'user', params: { id: 123 } }">
  User 123
</router-link>

<!-- With query -->
<router-link :to="{ path: '/search', query: { q: 'vue' } }">
  Search Vue
</router-link>

<!-- Active class -->
<router-link to="/" active-class="active" exact-active-class="exact-active">
  Home
</router-link>
```

### router-view

Placeholder where matched components render:

```vue
<template>
  <header>...</header>
  
  <!-- Current route's component renders here -->
  <router-view />
  
  <footer>...</footer>
</template>
```

## History Modes

### createWebHistory (HTML5 Mode)

```typescript
import { createWebHistory } from 'vue-router'

// URLs: /about, /users/123
const router = createRouter({
  history: createWebHistory(),
  routes
})
```

**Note**: Requires server configuration to handle all routes.

### createWebHashHistory (Hash Mode)

```typescript
import { createWebHashHistory } from 'vue-router'

// URLs: /#/about, /#/users/123
const router = createRouter({
  history: createWebHashHistory(),
  routes
})
```

No server configuration needed, but less clean URLs.

### createMemoryHistory (SSR/Testing)

```typescript
import { createMemoryHistory } from 'vue-router'

// No URL changes - good for testing
const router = createRouter({
  history: createMemoryHistory(),
  routes
})
```

## Project Structure

```
src/
├── router/
│   └── index.ts       # Router configuration
├── views/             # Route components (pages)
│   ├── Home.vue
│   ├── About.vue
│   └── users/
│       ├── UserList.vue
│       └── UserDetail.vue
└── components/        # Reusable components
    └── ...
```

**Convention**: Use `views/` for route components, `components/` for reusable pieces.

## Resources

- [Vue Router Documentation](https://router.vuejs.org/) — Official Vue Router documentation

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*