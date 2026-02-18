---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-middleware"
---

# Route Middleware

Middleware runs before navigating to a route, perfect for authentication and guards.

## Creating Middleware

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const { loggedIn } = useUserSession()
  
  if (!loggedIn.value) {
    return navigateTo('/login')
  }
})
```

## Using Middleware

### Per-Page

```vue
<script setup>
definePageMeta({
  middleware: 'auth'
})
</script>
```

### Multiple Middleware

```vue
<script setup>
definePageMeta({
  middleware: ['auth', 'admin']
})
</script>
```

### Global Middleware

```typescript
// middleware/analytics.global.ts
export default defineNuxtRouteMiddleware((to) => {
  // Runs on every route change
  trackPageView(to.fullPath)
})
```

## Auth Middleware Example

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  const { status, data } = useAuth()
  
  // Skip if already on login page
  if (to.path === '/login') return
  
  // Redirect to login if not authenticated
  if (status.value !== 'authenticated') {
    return navigateTo('/login', {
      replace: true
    })
  }
})

// middleware/admin.ts
export default defineNuxtRouteMiddleware(() => {
  const { data } = useAuth()
  
  if (data.value?.role !== 'admin') {
    throw createError({
      statusCode: 403,
      message: 'Forbidden'
    })
  }
})
```

## Inline Middleware

```vue
<script setup>
definePageMeta({
  middleware: [
    function (to, from) {
      // Custom inline logic
      if (to.params.id === '0') {
        return abortNavigation('Invalid ID')
      }
    }
  ]
})
</script>
```

## Navigation Results

```typescript
export default defineNuxtRouteMiddleware((to, from) => {
  // Allow navigation (default)
  return
  
  // Redirect
  return navigateTo('/other-page')
  
  // Redirect with options
  return navigateTo('/login', {
    replace: true,
    redirectCode: 301
  })
  
  // External redirect
  return navigateTo('https://nuxt.com', {
    external: true
  })
  
  // Abort with error
  return abortNavigation('Not allowed')
  
  // Abort with error object
  return abortNavigation(
    createError({ statusCode: 404, message: 'Not Found' })
  )
})
```

## Async Middleware

```typescript
// middleware/subscription.ts
export default defineNuxtRouteMiddleware(async (to) => {
  const { data: subscription } = await useFetch('/api/subscription')
  
  if (!subscription.value?.active) {
    return navigateTo('/subscribe')
  }
})
```

## Resources

- [Route Middleware](https://nuxt.com/docs/guide/directory-structure/middleware) — Nuxt middleware documentation

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*