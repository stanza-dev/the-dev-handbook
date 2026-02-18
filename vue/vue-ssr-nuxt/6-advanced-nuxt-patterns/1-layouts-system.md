---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-layouts-system"
---

# Layouts System

Layouts wrap your pages to provide consistent structure like headers, footers, and navigation.

## Default Layout

```vue
<!-- layouts/default.vue -->
<template>
  <div class="layout">
    <AppHeader />
    <main class="content">
      <slot />  <!-- Page content goes here -->
    </main>
    <AppFooter />
  </div>
</template>

<style scoped>
.layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}
.content {
  flex: 1;
  padding: 2rem;
}
</style>
```

## Using Layouts in App

```vue
<!-- app.vue -->
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

## Custom Layouts

```vue
<!-- layouts/dashboard.vue -->
<template>
  <div class="dashboard-layout">
    <Sidebar />
    <div class="dashboard-main">
      <DashboardHeader />
      <slot />
    </div>
  </div>
</template>
```

Use in page:

```vue
<!-- pages/dashboard/index.vue -->
<script setup>
definePageMeta({
  layout: 'dashboard'
})
</script>

<template>
  <h1>Dashboard Home</h1>
</template>
```

## Dynamic Layouts

```vue
<script setup>
const route = useRoute()

// Change layout based on condition
definePageMeta({
  layout: false  // Disable auto layout
})

const layoutName = computed(() => {
  return route.meta.isAdmin ? 'admin' : 'default'
})
</script>

<template>
  <NuxtLayout :name="layoutName">
    <h1>Dynamic Layout Page</h1>
  </NuxtLayout>
</template>
```

## Layout Transitions

```vue
<!-- app.vue -->
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>

<style>
.layout-enter-active,
.layout-leave-active {
  transition: opacity 0.3s;
}
.layout-enter-from,
.layout-leave-to {
  opacity: 0;
}
</style>
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    layoutTransition: { name: 'layout', mode: 'out-in' }
  }
})
```

## Named Slots in Layouts

```vue
<!-- layouts/with-sidebar.vue -->
<template>
  <div class="layout-with-sidebar">
    <aside class="sidebar">
      <slot name="sidebar" />
    </aside>
    <main class="main">
      <slot />
    </main>
  </div>
</template>
```

```vue
<!-- pages/docs.vue -->
<script setup>
definePageMeta({ layout: 'with-sidebar' })
</script>

<template>
  <NuxtLayout>
    <template #sidebar>
      <nav>
        <NuxtLink to="/docs/intro">Introduction</NuxtLink>
        <NuxtLink to="/docs/setup">Setup</NuxtLink>
      </nav>
    </template>
    
    <h1>Documentation Content</h1>
  </NuxtLayout>
</template>
```

## Resources

- [Nuxt Layouts](https://nuxt.com/docs/guide/directory-structure/layouts) — Nuxt layouts documentation

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*