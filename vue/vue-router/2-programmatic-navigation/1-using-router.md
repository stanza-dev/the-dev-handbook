---
source_course: "vue-router"
source_lesson: "vue-router-using-router"
---

# Navigating with useRouter

While `<router-link>` is great for declarative navigation, you often need to navigate programmatically in response to user actions or async operations.

## useRouter vs useRoute

```typescript
import { useRouter, useRoute } from 'vue-router'

// useRouter - for NAVIGATION (methods)
const router = useRouter()

// useRoute - for READING current route (reactive)
const route = useRoute()
```

## router.push()

Navigate to a new URL and add to history:

```vue
<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

// String path
function goHome() {
  router.push('/')
}

// Path object
function goToUser(id: number) {
  router.push({ path: `/users/${id}` })
}

// Named route with params
function goToUserNamed(id: number) {
  router.push({ name: 'user', params: { id } })
}

// With query parameters
function search(query: string) {
  router.push({ 
    path: '/search', 
    query: { q: query, page: 1 } 
  })
}

// With hash
function goToSection() {
  router.push({ path: '/docs', hash: '#installation' })
}
</script>
```

### push() Returns a Promise

```typescript
async function handleSubmit() {
  await saveData()
  
  try {
    await router.push('/success')
    console.log('Navigation successful')
  } catch (error) {
    // Navigation was prevented (by a guard)
    console.error('Navigation failed:', error)
  }
}
```

## router.replace()

Navigate without adding history entry:

```typescript
// User can't go back to previous page
router.replace('/login')

// Same options as push()
router.replace({ name: 'dashboard' })

// With push option
router.push({ path: '/new-page', replace: true })
```

Use cases:
- After login (don't want to go back to login page)
- Redirects
- URL corrections

## router.go()

Navigate through history:

```typescript
// Go back one page (like browser back button)
router.go(-1)

// Go forward one page
router.go(1)

// Go back 3 pages
router.go(-3)

// Convenience aliases
router.back()     // same as router.go(-1)
router.forward()  // same as router.go(1)
```

## Common Navigation Patterns

### After Form Submission

```vue
<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()
const form = reactive({ name: '', email: '' })

async function handleSubmit() {
  const result = await api.createUser(form)
  
  // Navigate to new user's page
  router.push({ 
    name: 'user', 
    params: { id: result.id } 
  })
}
</script>
```

### Conditional Navigation

```typescript
function proceedToCheckout() {
  if (!isLoggedIn.value) {
    router.push({
      name: 'login',
      query: { redirect: '/checkout' }  // Remember where to go after login
    })
  } else {
    router.push('/checkout')
  }
}
```

### Navigation After Async Operation

```typescript
async function deleteItem(id: number) {
  const confirmed = await showConfirmDialog('Delete this item?')
  
  if (confirmed) {
    await api.deleteItem(id)
    router.push({ name: 'items' })  // Go back to list
  }
}
```

### Redirect After Login

```typescript
async function login(credentials: Credentials) {
  await authStore.login(credentials)
  
  // Check for redirect query param
  const redirect = route.query.redirect as string
  router.push(redirect || '/dashboard')
}
```

## Important: params vs path

```typescript
// ❌ params are IGNORED when path is provided
router.push({ path: '/user', params: { id: 123 } })

// ✅ Use name with params
router.push({ name: 'user', params: { id: 123 } })

// ✅ Or include param in path
router.push({ path: `/user/${123}` })
```

## Waiting for Navigation

```typescript
import { onBeforeRouteLeave } from 'vue-router'

// In component
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    return confirm('You have unsaved changes. Leave anyway?')
  }
})
```

## Resources

- [Programmatic Navigation](https://router.vuejs.org/guide/essentials/navigation.html) — Official guide on programmatic navigation

---

> 📘 *This lesson is part of the [Vue Router: Complete Navigation Guide](https://stanza.dev/courses/vue-router) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*