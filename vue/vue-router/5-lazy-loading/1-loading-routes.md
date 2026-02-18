---
source_course: "vue-router"
source_lesson: "vue-router-lazy-loading-routes"
---

# Lazy Loading Route Components

Lazy loading splits your app into chunks that load on demand, improving initial load time.

## Basic Lazy Loading

Use dynamic imports:

```typescript
const routes = [
  {
    path: '/',
    // Loaded immediately
    component: Home
  },
  {
    path: '/about',
    // Loaded when route is visited
    component: () => import('@/views/About.vue')
  },
  {
    path: '/dashboard',
    // Loaded when route is visited
    component: () => import('@/views/Dashboard.vue')
  }
]
```

## Named Chunks

Group components into named bundles:

```typescript
const routes = [
  {
    path: '/user/:id',
    component: () => import(
      /* webpackChunkName: "user" */
      '@/views/User.vue'
    )
  },
  {
    path: '/user/:id/profile',
    component: () => import(
      /* webpackChunkName: "user" */
      '@/views/UserProfile.vue'
    )
  },
  {
    path: '/user/:id/posts',
    component: () => import(
      /* webpackChunkName: "user" */
      '@/views/UserPosts.vue'
    )
  }
]
// All user-related components bundled together
```

With Vite:

```typescript
// Vite uses Rollup, different comment syntax
const UserProfile = () => import('@/views/UserProfile.vue')
// Vite automatically names chunks based on file name
```

## Grouping Routes by Feature

```typescript
// Lazy load entire feature modules
const adminRoutes = [
  {
    path: '/admin',
    component: () => import('@/views/admin/Layout.vue'),
    children: [
      {
        path: 'users',
        component: () => import('@/views/admin/Users.vue')
      },
      {
        path: 'settings',
        component: () => import('@/views/admin/Settings.vue')
      }
    ]
  }
]
```

## Loading State

Show loading indicator while component loads:

```vue
<!-- App.vue -->
<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const isLoading = ref(false)

router.beforeEach(() => {
  isLoading.value = true
})

router.afterEach(() => {
  isLoading.value = false
})
</script>

<template>
  <div v-if="isLoading" class="loading-bar"></div>
  <router-view />
</template>
```

## Suspense with Async Components

```vue
<template>
  <router-view v-slot="{ Component }">
    <Suspense>
      <template #default>
        <component :is="Component" />
      </template>
      <template #fallback>
        <LoadingSpinner />
      </template>
    </Suspense>
  </router-view>
</template>
```

## Prefetching Routes

```typescript
// Prefetch on hover
function prefetchRoute(routeName: string) {
  const route = router.resolve({ name: routeName })
  const component = route.matched[0]?.components?.default
  
  if (typeof component === 'function') {
    component()  // Triggers the import
  }
}
```

```vue
<template>
  <router-link 
    to="/about" 
    @mouseenter="prefetchRoute('about')"
  >
    About
  </router-link>
</template>
```

## Error Handling for Lazy Components

```typescript
import { defineAsyncComponent } from 'vue'

const routes = [
  {
    path: '/dashboard',
    component: defineAsyncComponent({
      loader: () => import('@/views/Dashboard.vue'),
      loadingComponent: LoadingComponent,
      errorComponent: ErrorComponent,
      delay: 200,  // Show loading after 200ms
      timeout: 10000  // Timeout after 10s
    })
  }
]
```

## Route-Level Code Splitting Analysis

```bash
# Analyze your bundle
npx vite-bundle-analyzer

# Or with webpack
npx webpack-bundle-analyzer dist/stats.json
```

## Best Practices

```typescript
const routes = [
  // ✅ Eagerly load core routes
  {
    path: '/',
    component: Home  // No lazy loading for home
  },
  
  // ✅ Lazy load feature routes
  {
    path: '/settings',
    component: () => import('@/views/Settings.vue')
  },
  
  // ✅ Group related lazy routes
  {
    path: '/reports',
    component: () => import('@/views/reports/Layout.vue'),
    children: [
      {
        path: 'sales',
        component: () => import('@/views/reports/Sales.vue')
      },
      {
        path: 'inventory',
        component: () => import('@/views/reports/Inventory.vue')
      }
    ]
  },
  
  // ✅ Lazy load rarely accessed routes
  {
    path: '/admin',
    component: () => import('@/views/Admin.vue'),
    meta: { requiresAdmin: true }
  }
]
```

## Resources

- [Lazy Loading Routes](https://router.vuejs.org/guide/advanced/lazy-loading.html) — Official guide on route lazy loading

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*