---
source_course: "vue-router"
source_lesson: "vue-router-dynamic-routes"
---

# Dynamic Routes and Parameters

Dynamic routes allow you to match routes with variable segments, like user IDs or product slugs.

## Basic Dynamic Segments

Use `:paramName` syntax:

```typescript
const routes = [
  {
    path: '/users/:id',
    name: 'user',
    component: UserDetail
  },
  {
    path: '/posts/:slug',
    name: 'post',
    component: PostDetail
  }
]
```

Matches:
- `/users/123` → `{ id: '123' }`
- `/users/abc` → `{ id: 'abc' }`
- `/posts/my-first-post` → `{ slug: 'my-first-post' }`

## Accessing Route Params

### In Composition API

```vue
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()

// Access params
console.log(route.params.id)  // '123'
</script>

<template>
  <h1>User {{ route.params.id }}</h1>
</template>
```

### Reacting to Param Changes

When navigating between `/users/1` and `/users/2`, the component is **reused**:

```vue
<script setup>
import { watch } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const user = ref(null)

// Watch for param changes
watch(
  () => route.params.id,
  async (newId) => {
    user.value = await fetchUser(newId)
  },
  { immediate: true }
)
</script>
```

## Multiple Parameters

```typescript
const routes = [
  {
    // /users/123/posts/456
    path: '/users/:userId/posts/:postId',
    component: UserPost
  }
]
```

```vue
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()
// route.params = { userId: '123', postId: '456' }
</script>
```

## Optional Parameters

Add `?` to make a parameter optional:

```typescript
const routes = [
  {
    // Matches /users and /users/123
    path: '/users/:id?',
    component: Users
  }
]
```

## Repeatable Parameters

Use `+` or `*` for repeatable segments:

```typescript
const routes = [
  {
    // /files/one/two/three → params.path = ['one', 'two', 'three']
    path: '/files/:path+',  // At least one segment required
    component: FileExplorer
  },
  {
    // /docs or /docs/a/b/c
    path: '/docs/:sections*',  // Zero or more segments
    component: Documentation
  }
]
```

## Custom Regex Constraints

Restrict what params can match:

```typescript
const routes = [
  {
    // Only matches numeric IDs
    path: '/users/:id(\\d+)',
    component: UserDetail
  },
  {
    // Matches everything else
    path: '/users/:username',
    component: UserProfile
  }
]
```

Now `/users/123` goes to UserDetail, `/users/alice` goes to UserProfile.

## Named Routes

Always name your routes:

```typescript
const routes = [
  {
    path: '/users/:id',
    name: 'user-detail',  // Named route
    component: UserDetail
  }
]
```

```vue
<!-- Use name instead of path -->
<router-link :to="{ name: 'user-detail', params: { id: 123 } }">
  View User
</router-link>
```

Benefits:
- No hardcoded paths scattered in code
- Automatic encoding
- Safe refactoring

## Query Parameters

Query params are separate from route params:

```typescript
// URL: /search?q=vue&page=2

const route = useRoute()
console.log(route.query.q)     // 'vue'
console.log(route.query.page)  // '2' (always strings)
```

```vue
<!-- Link with query -->
<router-link :to="{ path: '/search', query: { q: 'vue', page: 2 } }">
  Search
</router-link>
```

## Props Mode

Pass route params as props (recommended):

```typescript
const routes = [
  {
    path: '/users/:id',
    component: UserDetail,
    props: true  // Pass params as props
  }
]
```

```vue
<!-- UserDetail.vue -->
<script setup>
defineProps<{
  id: string
}>()
</script>

<template>
  <h1>User {{ id }}</h1>
</template>
```

The component is now decoupled from the router!

## Resources

- [Dynamic Route Matching](https://router.vuejs.org/guide/essentials/dynamic-matching.html) — Official guide on dynamic routes

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*