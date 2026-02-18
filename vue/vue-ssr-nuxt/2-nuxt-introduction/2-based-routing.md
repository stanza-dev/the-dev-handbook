---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-file-based-routing"
---

# File-Based Routing

Nuxt automatically creates routes based on your file structure in the `pages/` directory.

## Basic Routes

```
pages/
├── index.vue        → /
├── about.vue        → /about
├── contact.vue      → /contact
└── pricing.vue      → /pricing
```

## Nested Routes

```
pages/
├── users/
│   ├── index.vue    → /users
│   └── profile.vue  → /users/profile
└── blog/
    ├── index.vue    → /blog
    └── [slug].vue   → /blog/:slug
```

## Dynamic Routes

Use square brackets for dynamic segments:

```
pages/
├── users/
│   ├── index.vue      → /users
│   └── [id].vue       → /users/:id
└── posts/
    └── [slug].vue     → /posts/:slug
```

```vue
<!-- pages/users/[id].vue -->
<script setup>
const route = useRoute()
const userId = route.params.id  // '123' from /users/123
</script>

<template>
  <h1>User {{ userId }}</h1>
</template>
```

## Catch-All Routes

```
pages/
└── [...slug].vue    → Matches any path
```

```vue
<!-- pages/[...slug].vue -->
<script setup>
const route = useRoute()
// /docs/a/b/c → params.slug = ['a', 'b', 'c']
console.log(route.params.slug)
</script>
```

## Nested Layouts with Parent

```
pages/
└── users/
    ├── index.vue       → /users
    └── [id]/
        ├── index.vue   → /users/:id
        ├── posts.vue   → /users/:id/posts
        └── settings.vue → /users/:id/settings
```

With parent layout:

```
pages/
└── users/
    ├── index.vue
    └── [id].vue         → Parent with <NuxtPage />
        └── index.vue    → Nested child
```

## Navigation

### NuxtLink Component

```vue
<template>
  <!-- Internal navigation -->
  <NuxtLink to="/">Home</NuxtLink>
  <NuxtLink to="/about">About</NuxtLink>
  
  <!-- With params -->
  <NuxtLink :to="{ path: '/users/' + user.id }">
    {{ user.name }}
  </NuxtLink>
  
  <!-- Named route -->
  <NuxtLink :to="{ name: 'users-id', params: { id: user.id } }">
    {{ user.name }}
  </NuxtLink>
  
  <!-- External link -->
  <NuxtLink to="https://vuejs.org" external>
    Vue.js
  </NuxtLink>
</template>
```

### Programmatic Navigation

```vue
<script setup>
const router = useRouter()

function goToUser(id: number) {
  router.push(`/users/${id}`)
}

function goBack() {
  router.back()
}
</script>
```

### navigateTo Utility

```vue
<script setup>
async function handleLogin() {
  await login()
  // Works in both client and server
  await navigateTo('/dashboard')
}

// With options
await navigateTo('/login', { 
  replace: true,
  redirectCode: 301 
})

// External URL
await navigateTo('https://nuxt.com', { 
  external: true 
})
</script>
```

## Page Metadata

```vue
<!-- pages/about.vue -->
<script setup>
definePageMeta({
  title: 'About Us',
  layout: 'default',
  middleware: 'auth',
  keepalive: true,
  
  // Custom metadata
  pageTransition: {
    name: 'page',
    mode: 'out-in'
  }
})

// Dynamic head
useHead({
  title: 'About Us | My App',
  meta: [
    { name: 'description', content: 'Learn about our company' }
  ]
})
</script>
```

## Resources

- [Nuxt Routing](https://nuxt.com/docs/getting-started/routing) — Nuxt file-based routing

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*