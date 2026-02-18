---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-suspense-component"
---

# Handling Async with Suspense

`<Suspense>` is a built-in component that handles async dependencies in the component tree, showing fallback content while waiting.

## Basic Suspense Usage

```vue
<template>
  <Suspense>
    <!-- Component with async setup -->
    <template #default>
      <AsyncComponent />
    </template>
    
    <!-- Shown while loading -->
    <template #fallback>
      <LoadingSpinner />
    </template>
  </Suspense>
</template>
```

## Async Setup in Components

Components with async operations in `<script setup>` trigger Suspense:

```vue
<!-- UserProfile.vue -->
<script setup lang="ts">
// Top-level await makes this an async component
const response = await fetch('/api/user')
const user = await response.json()
</script>

<template>
  <div class="profile">
    <h1>{{ user.name }}</h1>
    <p>{{ user.email }}</p>
  </div>
</template>
```

```vue
<!-- Parent -->
<template>
  <Suspense>
    <UserProfile />
    <template #fallback>
      <p>Loading user profile...</p>
    </template>
  </Suspense>
</template>
```

## Multiple Async Dependencies

Suspense waits for ALL async dependencies:

```vue
<template>
  <Suspense>
    <template #default>
      <div class="dashboard">
        <UserStats />      <!-- async -->
        <RecentActivity /> <!-- async -->
        <Notifications />  <!-- async -->
      </div>
    </template>
    
    <template #fallback>
      <DashboardSkeleton />
    </template>
  </Suspense>
</template>
```

## Error Handling with onErrorCaptured

```vue
<script setup lang="ts">
import { ref, onErrorCaptured } from 'vue'

const error = ref<Error | null>(null)

onErrorCaptured((err) => {
  error.value = err
  return false  // Prevent propagation
})
</script>

<template>
  <div v-if="error" class="error">
    <p>Something went wrong: {{ error.message }}</p>
    <button @click="error = null">Retry</button>
  </div>
  
  <Suspense v-else>
    <AsyncContent />
    <template #fallback>
      <LoadingSpinner />
    </template>
  </Suspense>
</template>
```

## Suspense Events

Listen to Suspense state changes:

```vue
<script setup>
function onPending() {
  console.log('Started loading')
}

function onResolve() {
  console.log('Finished loading')
}

function onFallback() {
  console.log('Fallback is now shown')
}
</script>

<template>
  <Suspense
    @pending="onPending"
    @resolve="onResolve"
    @fallback="onFallback"
  >
    <AsyncComponent />
    <template #fallback>
      <Loading />
    </template>
  </Suspense>
</template>
```

## Nested Suspense

Handle different loading states independently:

```vue
<template>
  <!-- Outer Suspense for layout -->
  <Suspense>
    <template #default>
      <AsyncLayout>
        <!-- Inner Suspense for content -->
        <Suspense>
          <template #default>
            <AsyncContent />
          </template>
          <template #fallback>
            <ContentSkeleton />
          </template>
        </Suspense>
      </AsyncLayout>
    </template>
    <template #fallback>
      <FullPageLoader />
    </template>
  </Suspense>
</template>
```

## Practical Example: Data Fetching Page

```vue
<!-- PostPage.vue -->
<script setup lang="ts">
import { ref, onErrorCaptured, Suspense } from 'vue'

const error = ref<Error | null>(null)
const key = ref(0)

onErrorCaptured((err) => {
  error.value = err
  return false
})

function retry() {
  error.value = null
  key.value++  // Force re-mount
}
</script>

<template>
  <div class="post-page">
    <div v-if="error" class="error-state">
      <h2>Failed to load</h2>
      <p>{{ error.message }}</p>
      <button @click="retry">Try Again</button>
    </div>
    
    <Suspense v-else :key="key">
      <template #default>
        <PostContent />
        <CommentSection />
        <RelatedPosts />
      </template>
      
      <template #fallback>
        <div class="loading-state">
          <PostSkeleton />
          <CommentsSkeleton />
        </div>
      </template>
    </Suspense>
  </div>
</template>
```

```vue
<!-- PostContent.vue -->
<script setup lang="ts">
const props = defineProps<{ postId: string }>()

// This await triggers Suspense in parent
const response = await fetch(`/api/posts/${props.postId}`)
if (!response.ok) throw new Error('Failed to fetch post')
const post = await response.json()
</script>

<template>
  <article>
    <h1>{{ post.title }}</h1>
    <div v-html="post.content"></div>
  </article>
</template>
```

## Important Notes

1. **Experimental Feature**: Suspense is still experimental in Vue 3
2. **SSR Support**: Works with server-side rendering
3. **Keep-Alive**: Compatible with `<KeepAlive>` for caching
4. **Transition**: Can wrap Suspense content with `<Transition>`

## Resources

- [Suspense](https://vuejs.org/guide/built-ins/suspense.html) — Official Vue documentation on Suspense

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*