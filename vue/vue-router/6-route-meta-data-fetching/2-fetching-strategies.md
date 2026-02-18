---
source_course: "vue-router"
source_lesson: "vue-router-data-fetching-strategies"
---

# Data Fetching Strategies

Decide when and how to fetch data for your routes.

## Strategy 1: Fetch After Navigation

Navigate first, show loading state:

```vue
<script setup>
import { ref, watchEffect } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const user = ref(null)
const loading = ref(true)
const error = ref(null)

watchEffect(async () => {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(`/api/users/${route.params.id}`)
    if (!response.ok) throw new Error('User not found')
    user.value = await response.json()
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <div v-if="loading">Loading...</div>
  <div v-else-if="error">{{ error }}</div>
  <div v-else>
    <h1>{{ user.name }}</h1>
  </div>
</template>
```

**Pros:** Fast navigation, progressive loading
**Cons:** Shows empty/loading state

## Strategy 2: Fetch Before Navigation

Wait for data, then navigate:

```typescript
const routes = [
  {
    path: '/users/:id',
    component: () => import('./UserProfile.vue'),
    beforeEnter: async (to) => {
      const userStore = useUserStore()
      
      try {
        await userStore.fetchUser(to.params.id)
      } catch (error) {
        return { name: 'not-found' }
      }
    }
  }
]
```

Or with props:

```typescript
const routes = [
  {
    path: '/posts/:id',
    component: PostDetail,
    props: true,
    beforeEnter: async (to) => {
      const response = await fetch(`/api/posts/${to.params.id}`)
      if (!response.ok) return { name: 'not-found' }
      to.params.post = await response.json()
      return true
    }
  }
]
```

**Pros:** Component always has data
**Cons:** Slower navigation, blocks UI

## Strategy 3: Suspense with Async Setup

```vue
<!-- UserProfile.vue -->
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()

// Async operation in setup
const response = await fetch(`/api/users/${route.params.id}`)
const user = await response.json()
</script>

<template>
  <h1>{{ user.name }}</h1>
</template>
```

```vue
<!-- App.vue -->
<template>
  <router-view v-slot="{ Component }">
    <Suspense>
      <component :is="Component" />
      
      <template #fallback>
        <div class="loading">Loading...</div>
      </template>
    </Suspense>
  </router-view>
</template>
```

## Navigation Feedback

Show loading during route transitions:

```vue
<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const isNavigating = ref(false)

router.beforeEach(() => {
  isNavigating.value = true
})

router.afterEach(() => {
  isNavigating.value = false
})

router.onError(() => {
  isNavigating.value = false
})
</script>

<template>
  <div class="app">
    <LoadingBar v-if="isNavigating" />
    <router-view />
  </div>
</template>
```

## Resources

- [Data Fetching](https://router.vuejs.org/guide/advanced/data-fetching.html) — Vue Router data fetching guide

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*