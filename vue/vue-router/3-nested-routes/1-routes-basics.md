---
source_course: "vue-router"
source_lesson: "vue-router-nested-routes-basics"
---

# Understanding Nested Routes

Nested routes allow you to create complex UI layouts where parts of the page change based on the URL while other parts remain constant.

## The Concept

```
/user/123           →  User layout + UserHome
/user/123/profile   →  User layout + UserProfile
/user/123/posts     →  User layout + UserPosts
```

The user layout (header, sidebar) stays the same while the content area changes.

## Basic Nested Routes

```typescript
const routes = [
  {
    path: '/user/:id',
    component: UserLayout,
    children: [
      {
        path: '',  // Matches /user/:id
        name: 'user-home',
        component: UserHome
      },
      {
        path: 'profile',  // Matches /user/:id/profile
        name: 'user-profile',
        component: UserProfile
      },
      {
        path: 'posts',  // Matches /user/:id/posts
        name: 'user-posts',
        component: UserPosts
      }
    ]
  }
]
```

**Note**: Child paths don't start with `/` - they're relative to parent.

## Parent Component with router-view

```vue
<!-- UserLayout.vue -->
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()
const userId = computed(() => route.params.id)
</script>

<template>
  <div class="user-layout">
    <header>
      <h1>User {{ userId }}</h1>
      <nav>
        <router-link :to="{ name: 'user-home', params: { id: userId } }">
          Home
        </router-link>
        <router-link :to="{ name: 'user-profile', params: { id: userId } }">
          Profile
        </router-link>
        <router-link :to="{ name: 'user-posts', params: { id: userId } }">
          Posts
        </router-link>
      </nav>
    </header>
    
    <main>
      <!-- Child routes render here -->
      <router-view />
    </main>
  </div>
</template>
```

## Multi-Level Nesting

```typescript
const routes = [
  {
    path: '/admin',
    component: AdminLayout,
    children: [
      {
        path: 'users',
        component: AdminUsersLayout,
        children: [
          { path: '', component: UserList },
          { path: ':id', component: UserDetail },
          { path: ':id/edit', component: UserEdit }
        ]
      },
      {
        path: 'settings',
        component: AdminSettings
      }
    ]
  }
]
```

## Empty Path Children

Use empty string for default child:

```typescript
{
  path: '/dashboard',
  component: DashboardLayout,
  children: [
    {
      path: '',  // Default when /dashboard is accessed
      name: 'dashboard-overview',
      component: DashboardOverview
    },
    {
      path: 'analytics',
      component: Analytics
    }
  ]
}
```

## Nested Routes Without Parent Component

Group routes without a layout component:

```typescript
{
  path: '/settings',
  // No component - children render in parent's router-view
  children: [
    { path: 'profile', component: ProfileSettings },
    { path: 'account', component: AccountSettings },
    { path: 'notifications', component: NotificationSettings }
  ]
}
```

## Accessing Parent Route Params

Child components can access parent params:

```vue
<!-- UserPosts.vue (child of /user/:id) -->
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()
// Parent's :id is accessible
const userId = computed(() => route.params.id)
</script>
```

## Redirecting to Default Child

```typescript
{
  path: '/user/:id',
  component: UserLayout,
  redirect: { name: 'user-profile' },  // Redirect to default child
  children: [
    {
      path: 'profile',
      name: 'user-profile',
      component: UserProfile
    },
    {
      path: 'settings',
      name: 'user-settings',
      component: UserSettings
    }
  ]
}
```

## Named router-view

Render multiple components at the same level:

```typescript
{
  path: '/dashboard',
  components: {
    default: DashboardMain,
    sidebar: DashboardSidebar,
    header: DashboardHeader
  }
}
```

```vue
<template>
  <router-view name="header" />
  <div class="layout">
    <router-view name="sidebar" />
    <router-view />  <!-- default -->
  </div>
</template>
```

## Resources

- [Nested Routes](https://router.vuejs.org/guide/essentials/nested-routes.html) — Official guide on nested routes

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*