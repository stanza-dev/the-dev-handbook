---
source_course: "vue-router"
source_lesson: "vue-router-global-guards"
---

# Global Navigation Guards

Navigation guards let you control navigation flow—redirect, cancel, or add logic before route changes.

## Guard Types Overview

| Guard | Scope | Use Case |
|-------|-------|----------|
| beforeEach | Global | Auth checks, logging |
| beforeResolve | Global | After async components |
| afterEach | Global | Analytics, page titles |
| beforeEnter | Per-route | Route-specific checks |
| beforeRouteEnter | Component | Fetch data before render |
| beforeRouteUpdate | Component | React to param changes |
| beforeRouteLeave | Component | Unsaved changes warning |

## router.beforeEach

Runs before every navigation:

```typescript
const router = createRouter({ /* ... */ })

router.beforeEach((to, from) => {
  console.log(`Navigating from ${from.path} to ${to.path}`)
  
  // to - target route
  // from - current route
  
  // Return values:
  // - nothing/true: continue navigation
  // - false: cancel navigation
  // - route location: redirect
})
```

### Authentication Guard

```typescript
router.beforeEach((to, from) => {
  const authStore = useAuthStore()
  
  // Check if route requires auth
  if (to.meta.requiresAuth && !authStore.isLoggedIn) {
    // Redirect to login with return URL
    return {
      name: 'login',
      query: { redirect: to.fullPath }
    }
  }
  
  // Check role-based access
  if (to.meta.requiredRole) {
    if (!authStore.hasRole(to.meta.requiredRole)) {
      return { name: 'unauthorized' }
    }
  }
})
```

### Set Page Title

```typescript
router.beforeEach((to) => {
  document.title = to.meta.title 
    ? `${to.meta.title} | My App` 
    : 'My App'
})
```

## router.afterEach

Runs after navigation completes (can't affect navigation):

```typescript
router.afterEach((to, from, failure) => {
  // Analytics tracking
  analytics.pageView(to.path)
  
  // Check if navigation failed
  if (failure) {
    console.log('Navigation failed:', failure)
  }
  
  // Scroll to top (alternative to scrollBehavior)
  window.scrollTo(0, 0)
})
```

## router.beforeResolve

Runs after all async route components are resolved:

```typescript
router.beforeResolve(async (to) => {
  // Good for fetching data that the route depends on
  if (to.meta.requiresData) {
    try {
      await fetchRequiredData(to.params)
    } catch (error) {
      // Redirect to error page
      return { name: 'error' }
    }
  }
})
```

## Async Guards

Guards can be async:

```typescript
router.beforeEach(async (to) => {
  // Wait for auth check
  const canAccess = await checkAccess(to)
  
  if (!canAccess) {
    return '/login'
  }
})
```

## Multiple Guards

Guards run in order:

```typescript
// First guard
router.beforeEach((to) => {
  console.log('Guard 1')
  // Continue to next guard
})

// Second guard
router.beforeEach((to) => {
  console.log('Guard 2')
  return false  // Cancels navigation
})

// Third guard (never runs if Guard 2 cancels)
router.beforeEach((to) => {
  console.log('Guard 3')
})
```

## Route Meta Fields

Attach custom data to routes:

```typescript
const routes = [
  {
    path: '/admin',
    component: Admin,
    meta: {
      requiresAuth: true,
      requiredRole: 'admin',
      title: 'Admin Dashboard',
      transition: 'slide'
    }
  }
]

// Access in guards
router.beforeEach((to) => {
  if (to.meta.requiresAuth) {
    // Check auth
  }
})

// Nested routes inherit meta
router.beforeEach((to) => {
  // Check all matched routes' meta
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // At least one parent/child requires auth
  }
})
```

## TypeScript: Typing Meta Fields

```typescript
// router.d.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    requiredRole?: string
    title?: string
    transition?: string
  }
}
```

Now TypeScript knows about your meta fields!

## Resources

- [Navigation Guards](https://router.vuejs.org/guide/advanced/navigation-guards.html) — Complete guide to navigation guards

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*