---
source_course: "vue-router"
source_lesson: "vue-router-route-meta"
---

# Route Meta Fields

Route meta fields allow you to attach custom data to routes for guards, breadcrumbs, titles, and more.

## Defining Meta Fields

```typescript
const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: Home,
    meta: {
      title: 'Home',
      requiresAuth: false,
      transition: 'fade'
    }
  },
  {
    path: '/dashboard',
    component: Dashboard,
    meta: {
      title: 'Dashboard',
      requiresAuth: true,
      roles: ['user', 'admin']
    }
  },
  {
    path: '/admin',
    component: AdminPanel,
    meta: {
      title: 'Admin Panel',
      requiresAuth: true,
      roles: ['admin'],
      layout: 'admin'
    }
  }
]
```

## Accessing Meta in Components

```vue
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()

// Access current route's meta
console.log(route.meta.title)
console.log(route.meta.requiresAuth)
</script>
```

## Using Meta in Guards

```typescript
router.beforeEach((to, from) => {
  // Check auth requirement
  if (to.meta.requiresAuth) {
    const auth = useAuthStore()
    if (!auth.isAuthenticated) {
      return { name: 'login', query: { redirect: to.fullPath } }
    }
    
    // Check roles
    if (to.meta.roles && !to.meta.roles.includes(auth.user?.role)) {
      return { name: 'forbidden' }
    }
  }
})
```

## Dynamic Document Title

```typescript
router.afterEach((to) => {
  const title = to.meta.title as string | undefined
  document.title = title ? `${title} | My App` : 'My App'
})
```

## TypeScript Support

```typescript
// Extend RouteMeta interface
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    title?: string
    requiresAuth?: boolean
    roles?: ('user' | 'admin' | 'moderator')[]
    layout?: 'default' | 'admin' | 'auth'
    transition?: string
    breadcrumb?: string | ((route: RouteLocationNormalized) => string)
  }
}
```

## Breadcrumbs from Meta

```vue
<script setup>
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

const breadcrumbs = computed(() => {
  const crumbs = []
  
  route.matched.forEach((record) => {
    if (record.meta.breadcrumb) {
      const label = typeof record.meta.breadcrumb === 'function'
        ? record.meta.breadcrumb(route)
        : record.meta.breadcrumb
      
      crumbs.push({
        label,
        path: record.path
      })
    }
  })
  
  return crumbs
})
</script>

<template>
  <nav class="breadcrumbs">
    <router-link to="/">Home</router-link>
    <template v-for="crumb in breadcrumbs" :key="crumb.path">
      <span> / </span>
      <router-link :to="crumb.path">{{ crumb.label }}</router-link>
    </template>
  </nav>
</template>
```

## Resources

- [Route Meta Fields](https://router.vuejs.org/guide/advanced/meta.html) — Vue Router meta fields documentation

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*