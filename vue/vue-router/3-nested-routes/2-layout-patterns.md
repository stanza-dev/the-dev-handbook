---
source_course: "vue-router"
source_lesson: "vue-router-layout-patterns"
---

# Layout Patterns

Real applications need different layouts for different sections. Here's how to implement them elegantly.

## Multiple Layouts

```typescript
const routes = [
  // Public layout (marketing pages)
  {
    path: '/',
    component: PublicLayout,
    children: [
      { path: '', component: Home },
      { path: 'about', component: About },
      { path: 'pricing', component: Pricing }
    ]
  },
  
  // Auth layout (login, signup)
  {
    path: '/auth',
    component: AuthLayout,
    children: [
      { path: 'login', component: Login },
      { path: 'signup', component: Signup },
      { path: 'forgot-password', component: ForgotPassword }
    ]
  },
  
  // Dashboard layout (authenticated)
  {
    path: '/app',
    component: DashboardLayout,
    meta: { requiresAuth: true },
    children: [
      { path: '', component: Dashboard },
      { path: 'settings', component: Settings },
      { path: 'profile', component: Profile }
    ]
  }
]
```

## Layout Components

```vue
<!-- layouts/PublicLayout.vue -->
<template>
  <div class="public-layout">
    <PublicHeader />
    <main>
      <router-view />
    </main>
    <PublicFooter />
  </div>
</template>

<!-- layouts/DashboardLayout.vue -->
<template>
  <div class="dashboard-layout">
    <DashboardSidebar />
    <div class="main-content">
      <DashboardHeader />
      <main>
        <router-view />
      </main>
    </div>
  </div>
</template>

<!-- layouts/AuthLayout.vue -->
<template>
  <div class="auth-layout">
    <div class="auth-card">
      <Logo />
      <router-view />
    </div>
  </div>
</template>
```

## Per-Route Layout Component

Using route meta:

```typescript
// routes
{
  path: '/login',
  component: Login,
  meta: { layout: 'auth' }
},
{
  path: '/dashboard',
  component: Dashboard,
  meta: { layout: 'dashboard' }
}
```

```vue
<!-- App.vue -->
<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import PublicLayout from './layouts/Public.vue'
import AuthLayout from './layouts/Auth.vue'
import DashboardLayout from './layouts/Dashboard.vue'

const route = useRoute()

const layouts = {
  public: PublicLayout,
  auth: AuthLayout,
  dashboard: DashboardLayout
}

const currentLayout = computed(() => 
  layouts[route.meta.layout as string] || PublicLayout
)
</script>

<template>
  <component :is="currentLayout">
    <router-view />
  </component>
</template>
```

## Transitions Between Routes

```vue
<template>
  <router-view v-slot="{ Component, route }">
    <Transition :name="route.meta.transition || 'fade'" mode="out-in">
      <component :is="Component" :key="route.path" />
    </Transition>
  </router-view>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter-from {
  transform: translateX(100%);
}

.slide-leave-to {
  transform: translateX(-100%);
}
</style>
```

## Keep-Alive with Routes

Cache route components:

```vue
<router-view v-slot="{ Component }">
  <KeepAlive :include="['Dashboard', 'Settings']">
    <component :is="Component" />
  </KeepAlive>
</router-view>
```

```vue
<!-- Or based on route meta -->
<router-view v-slot="{ Component, route }">
  <KeepAlive>
    <component 
      :is="Component" 
      :key="route.meta.keepAlive ? undefined : route.path" 
    />
  </KeepAlive>
</router-view>
```

## Passing Props to Layout

```vue
<template>
  <router-view v-slot="{ Component }">
    <DashboardLayout :sidebar-collapsed="isCollapsed">
      <component :is="Component" />
    </DashboardLayout>
  </router-view>
</template>
```

## Nested Named Views

```typescript
{
  path: '/dashboard',
  components: {
    default: DashboardMain,
    sidebar: DashboardSidebar
  },
  children: [
    {
      path: 'stats',
      components: {
        default: Stats,
        sidebar: StatsSidebar  // Override sidebar for this child
      }
    }
  ]
}
```

## Resources

- [Named Views](https://router.vuejs.org/guide/essentials/named-views.html) — Using multiple router-views

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*