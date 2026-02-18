---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-usefetch-useasyncdata"
---

# useFetch and useAsyncData

Nuxt provides composables for data fetching that work seamlessly with SSR.

## useFetch

The primary way to fetch data in Nuxt:

```vue
<script setup>
const { data, pending, error, refresh } = await useFetch('/api/users')
</script>

<template>
  <div v-if="pending">Loading...</div>
  <div v-else-if="error">Error: {{ error.message }}</div>
  <ul v-else>
    <li v-for="user in data" :key="user.id">{{ user.name }}</li>
  </ul>
</template>
```

## useFetch Options

```vue
<script setup>
const { data } = await useFetch('/api/users', {
  // HTTP method
  method: 'POST',
  
  // Request body
  body: { name: 'John' },
  
  // Query parameters
  query: { page: 1, limit: 10 },
  
  // Headers
  headers: { 'Authorization': 'Bearer token' },
  
  // Transform response
  transform: (response) => response.data,
  
  // Pick specific fields
  pick: ['id', 'name', 'email'],
  
  // Default value
  default: () => [],
  
  // Key for caching
  key: 'users-list',
  
  // Server only (no client fetch)
  server: true,
  
  // Lazy (don't block navigation)
  lazy: true,
  
  // Immediate (fetch on mount)
  immediate: true,
  
  // Watch sources to refetch
  watch: [page, filter]
})
</script>
```

## useAsyncData

For custom async operations:

```vue
<script setup>
const { data: user } = await useAsyncData('user', () => {
  return $fetch(`/api/users/${route.params.id}`)
})

// With options
const { data: posts } = await useAsyncData(
  'posts',
  () => fetchPosts(),
  {
    transform: (posts) => posts.filter(p => p.published),
    default: () => []
  }
)
</script>
```

## Lazy Fetching

Don't block navigation while fetching:

```vue
<script setup>
// useLazyFetch - doesn't block navigation
const { data, pending } = useLazyFetch('/api/posts')

// useLazyAsyncData
const { data: comments } = useLazyAsyncData('comments', fetchComments)
</script>

<template>
  <div v-if="pending">Loading posts...</div>
  <PostList v-else :posts="data" />
</template>
```

## Refreshing Data

```vue
<script setup>
const { data, refresh, pending } = await useFetch('/api/users')

// Manual refresh
async function handleRefresh() {
  await refresh()
}

// Clear cache and refetch
await refresh({ dedupe: true })
</script>

<template>
  <button @click="handleRefresh" :disabled="pending">
    {{ pending ? 'Refreshing...' : 'Refresh' }}
  </button>
</template>
```

## Watching for Changes

```vue
<script setup>
const page = ref(1)
const category = ref('all')

const { data: products } = await useFetch('/api/products', {
  query: { page, category },
  // Refetch when these change
  watch: [page, category]
})
</script>

<template>
  <select v-model="category">
    <option value="all">All</option>
    <option value="electronics">Electronics</option>
  </select>
  
  <button @click="page++">Next Page</button>
</template>
```

## Error Handling

```vue
<script setup>
const { data, error } = await useFetch('/api/users')

if (error.value) {
  // Handle error
  throw createError({
    statusCode: 404,
    message: 'Users not found'
  })
}
</script>

<template>
  <div v-if="error">
    <h2>Error</h2>
    <p>{{ error.message }}</p>
    <pre>{{ error.data }}</pre>
  </div>
</template>
```

## $fetch for Non-Reactive Calls

```vue
<script setup>
// For event handlers or non-reactive needs
async function createUser(userData: UserData) {
  const user = await $fetch('/api/users', {
    method: 'POST',
    body: userData
  })
  return user
}

async function deleteUser(id: number) {
  await $fetch(`/api/users/${id}`, {
    method: 'DELETE'
  })
}
</script>
```

## Caching Behavior

```vue
<script setup>
// Same key = shared data
const { data: user1 } = await useFetch('/api/user/1', { key: 'user-1' })
const { data: user2 } = await useFetch('/api/user/1', { key: 'user-1' })
// user1 and user2 share the same cached data

// Force fresh fetch
const { data: freshUser } = await useFetch('/api/user/1', {
  key: 'user-1-fresh',
  getCachedData: () => null  // Bypass cache
})
</script>
```

## Resources

- [Data Fetching](https://nuxt.com/docs/getting-started/data-fetching) — Nuxt data fetching guide

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*