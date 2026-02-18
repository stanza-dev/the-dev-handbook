---
source_course: "vue-router"
source_lesson: "vue-router-component-guards"
---

# Route and Component Guards

Beyond global guards, you can define guards on specific routes or within components.

## Per-Route Guards (beforeEnter)

Define guards directly on route config:

```typescript
const routes = [
  {
    path: '/admin',
    component: Admin,
    beforeEnter: (to, from) => {
      // Only for this specific route
      if (!isAdmin()) {
        return '/unauthorized'
      }
    }
  },
  {
    path: '/user/:id',
    component: User,
    // Array of guards
    beforeEnter: [checkAuth, checkOwnership, logAccess]
  }
]

// Reusable guard functions
function checkAuth(to, from) {
  if (!isLoggedIn()) return '/login'
}

function checkOwnership(to, from) {
  if (!canAccessUser(to.params.id)) return '/unauthorized'
}

function logAccess(to) {
  console.log('Accessing user:', to.params.id)
}
```

## Component Guards (Composition API)

### onBeforeRouteLeave

Warn about unsaved changes:

```vue
<script setup>
import { ref } from 'vue'
import { onBeforeRouteLeave } from 'vue-router'

const hasUnsavedChanges = ref(false)

onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    const answer = window.confirm(
      'You have unsaved changes. Leave anyway?'
    )
    if (!answer) return false  // Cancel navigation
  }
})
</script>
```

### onBeforeRouteUpdate

React when params change (component reused):

```vue
<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, onBeforeRouteUpdate } from 'vue-router'

const route = useRoute()
const user = ref(null)

async function fetchUser(id: string) {
  user.value = await api.getUser(id)
}

onMounted(() => {
  fetchUser(route.params.id as string)
})

// Called when /user/1 → /user/2
onBeforeRouteUpdate(async (to, from) => {
  if (to.params.id !== from.params.id) {
    await fetchUser(to.params.id as string)
  }
})
</script>
```

## Component Guards (Options API)

```javascript
export default {
  beforeRouteEnter(to, from, next) {
    // Called BEFORE component is created
    // No access to 'this'
    next(vm => {
      // Access component via 'vm'
      vm.initializeFromRoute(to)
    })
  },
  
  beforeRouteUpdate(to, from) {
    // Called when route changes but component is reused
    // Has access to 'this'
    this.fetchData(to.params.id)
  },
  
  beforeRouteLeave(to, from) {
    // Called when leaving this route
    // Has access to 'this'
    if (this.hasUnsavedChanges) {
      return confirm('Leave without saving?')
    }
  }
}
```

## Guard Resolution Order

1. `beforeRouteLeave` (leaving component)
2. Global `beforeEach`
3. `beforeRouteUpdate` (reused component)
4. Route config `beforeEnter`
5. Resolve async route components
6. `beforeRouteEnter` (entering component)
7. Global `beforeResolve`
8. Navigation confirmed
9. Global `afterEach`
10. DOM updates triggered
11. `next` callbacks in `beforeRouteEnter`

## Common Patterns

### Confirm Leaving with Form Changes

```vue
<script setup>
import { ref, computed } from 'vue'
import { onBeforeRouteLeave } from 'vue-router'

const originalData = ref({ name: '', email: '' })
const formData = ref({ name: '', email: '' })

const isDirty = computed(() => 
  JSON.stringify(formData.value) !== JSON.stringify(originalData.value)
)

onBeforeRouteLeave(() => {
  if (isDirty.value) {
    return confirm('Discard unsaved changes?')
  }
})
</script>
```

### Fetch Data Before Navigation

```typescript
const routes = [
  {
    path: '/post/:id',
    component: Post,
    beforeEnter: async (to) => {
      try {
        // Prefetch data
        const post = await api.getPost(to.params.id)
        // Store for component to use
        to.meta.post = post
      } catch (error) {
        return { name: 'not-found' }
      }
    }
  }
]
```

### Role-Based Route Protection

```typescript
function requireRole(...roles: string[]) {
  return (to, from) => {
    const authStore = useAuthStore()
    
    if (!authStore.isLoggedIn) {
      return { name: 'login', query: { redirect: to.fullPath } }
    }
    
    if (!roles.some(role => authStore.hasRole(role))) {
      return { name: 'unauthorized' }
    }
  }
}

// Usage
{
  path: '/admin',
  component: Admin,
  beforeEnter: requireRole('admin', 'superadmin')
}
```

## Resources

- [In-Component Guards](https://router.vuejs.org/guide/advanced/navigation-guards.html#In-Component-Guards) — Component-level navigation guards

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*