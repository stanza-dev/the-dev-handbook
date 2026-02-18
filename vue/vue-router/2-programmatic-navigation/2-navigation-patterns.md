---
source_course: "vue-router"
source_lesson: "vue-router-navigation-patterns"
---

# Advanced Navigation Patterns

Let's explore sophisticated navigation patterns used in real applications.

## Router Typing with TypeScript

```typescript
// router/index.ts
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/users/:id',
    name: 'user',
    component: () => import('@/views/User.vue'),
    meta: {
      requiresAuth: true,
      title: 'User Profile'
    }
  }
]

// Type-safe route names
type RouteNames = 'home' | 'user' | 'settings'

// Custom typed push function
function typedPush(name: RouteNames, params?: Record<string, string>) {
  return router.push({ name, params })
}
```

## Preserving Query Parameters

```typescript
function navigatePreservingQuery(path: string) {
  router.push({
    path,
    query: route.query  // Keep existing query params
  })
}

// Add to existing query
function addQueryParam(key: string, value: string) {
  router.push({
    query: {
      ...route.query,
      [key]: value
    }
  })
}

// Remove a query param
function removeQueryParam(key: string) {
  const { [key]: removed, ...rest } = route.query
  router.push({ query: rest })
}
```

## Navigation with State

Pass state without showing in URL:

```typescript
// Navigate with state
router.push({
  name: 'user',
  params: { id: 123 },
  state: { fromNotification: true }  // Hidden state
})

// Access in destination
const state = history.state
if (state?.fromNotification) {
  showWelcomeMessage()
}
```

## Scroll Behavior

Control scroll position on navigation:

```typescript
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    // Back/forward navigation: restore position
    if (savedPosition) {
      return savedPosition
    }
    
    // Navigate to hash anchor
    if (to.hash) {
      return {
        el: to.hash,
        behavior: 'smooth'
      }
    }
    
    // Same route navigation: don't scroll
    if (to.path === from.path) {
      return false
    }
    
    // Default: scroll to top
    return { top: 0, behavior: 'smooth' }
  }
})
```

### Delayed Scroll (Wait for Content)

```typescript
scrollBehavior(to, from, savedPosition) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ top: 0 })
    }, 500)  // Wait for transitions
  })
}
```

## Route Aliases

Multiple URLs for the same component:

```typescript
const routes = [
  {
    path: '/users',
    component: Users,
    alias: ['/people', '/members']  // All show Users component
  },
  {
    path: '/home',
    component: Home,
    alias: '/'  // Both / and /home show Home
  }
]
```

## Redirects

```typescript
const routes = [
  // Simple redirect
  {
    path: '/old-page',
    redirect: '/new-page'
  },
  
  // Named route redirect
  {
    path: '/user/:id/profile',
    redirect: { name: 'user' }  // Keeps params
  },
  
  // Function redirect (dynamic)
  {
    path: '/search/:query',
    redirect: (to) => {
      return {
        path: '/results',
        query: { q: to.params.query }
      }
    }
  },
  
  // Redirect all unknown routes
  {
    path: '/:pathMatch(.*)*',
    redirect: '/404'
  }
]
```

## Navigation Failures

Handle navigation failures gracefully:

```typescript
import { NavigationFailureType, isNavigationFailure } from 'vue-router'

async function navigate() {
  const failure = await router.push('/dashboard')
  
  if (failure) {
    if (isNavigationFailure(failure, NavigationFailureType.aborted)) {
      // Navigation was aborted (by a guard)
      console.log('Navigation aborted')
    }
    if (isNavigationFailure(failure, NavigationFailureType.duplicated)) {
      // Already on that route
      console.log('Already there')
    }
  }
}
```

## Router Ready State

Wait for initial navigation:

```typescript
// main.ts
router.isReady().then(() => {
  app.mount('#app')
})

// Or in a component
await router.isReady()
console.log('Router is ready')
```

## Resources

- [Scroll Behavior](https://router.vuejs.org/guide/advanced/scroll-behavior.html) — Controlling scroll on navigation

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*