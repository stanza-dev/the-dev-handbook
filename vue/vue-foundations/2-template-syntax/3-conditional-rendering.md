---
source_course: "vue-foundations"
source_lesson: "vue-foundations-conditional-rendering"
---

# Conditional Rendering with v-if and v-show

Vue provides two ways to conditionally render elements: `v-if` for conditional creation/destruction and `v-show` for conditional visibility. Understanding when to use each is essential.

## v-if Directive

The `v-if` directive conditionally renders an element based on a truthy value:

```vue
<script setup>
import { ref } from 'vue'

const isLoggedIn = ref(false)
const hasPermission = ref(true)
</script>

<template>
  <div v-if="isLoggedIn">
    <p>Welcome back!</p>
    <button>Logout</button>
  </div>
  
  <div v-if="hasPermission">
    <p>You have access to this content.</p>
  </div>
</template>
```

When `isLoggedIn` is `false`, the element and all its children are **completely removed** from the DOM.

## v-else and v-else-if

Provide alternative blocks when the condition is false:

```vue
<script setup>
import { ref } from 'vue'

const userRole = ref('admin')
const isAuthenticated = ref(true)
</script>

<template>
  <!-- Simple if/else -->
  <div v-if="isAuthenticated">
    <p>Welcome to your dashboard!</p>
  </div>
  <div v-else>
    <p>Please log in to continue.</p>
    <button>Login</button>
  </div>
  
  <!-- Multiple conditions -->
  <div v-if="userRole === 'admin'">
    <h2>Admin Panel</h2>
    <p>You have full access.</p>
  </div>
  <div v-else-if="userRole === 'editor'">
    <h2>Editor Dashboard</h2>
    <p>You can edit content.</p>
  </div>
  <div v-else-if="userRole === 'viewer'">
    <h2>Viewer Mode</h2>
    <p>You can view content.</p>
  </div>
  <div v-else>
    <h2>Guest</h2>
    <p>Please sign up for access.</p>
  </div>
</template>
```

**Important**: `v-else` and `v-else-if` must immediately follow a `v-if` or `v-else-if` element.

## v-if on `<template>`

To conditionally render multiple elements without a wrapper, use `v-if` on a `<template>` element:

```vue
<script setup>
import { ref } from 'vue'

const showDetails = ref(true)
</script>

<template>
  <template v-if="showDetails">
    <h2>Product Details</h2>
    <p>Description goes here...</p>
    <p>Price: $99.99</p>
  </template>
  <!-- No wrapper div in the final output! -->
</template>
```

The `<template>` element acts as an invisible wrapper that won't appear in the rendered HTML.

## v-show Directive

`v-show` toggles the element's CSS `display` property:

```vue
<script setup>
import { ref } from 'vue'

const isVisible = ref(true)
</script>

<template>
  <div v-show="isVisible">
    This element is always in the DOM,
    but may be hidden with display: none
  </div>
  
  <button @click="isVisible = !isVisible">
    Toggle Visibility
  </button>
</template>
```

With `v-show`, the element is **always rendered** and stays in the DOM. Only the `display` CSS property is toggled.

## v-if vs v-show: When to Use Which?

| Aspect | v-if | v-show |
|--------|------|--------|
| **DOM** | Adds/removes elements | Always in DOM |
| **Initial render** | Lazy (skipped if false) | Always rendered |
| **Toggle cost** | Higher (recreates elements) | Lower (CSS only) |
| **Best for** | Conditions that rarely change | Frequently toggled content |

### Use `v-if` when:

```vue
<script setup>
import { ref } from 'vue'

const userType = ref<'free' | 'premium'>('free')
</script>

<template>
  <!-- Condition unlikely to change frequently -->
  <PremiumFeatures v-if="userType === 'premium'" />
  <FreeFeatures v-else />
</template>
```

### Use `v-show` when:

```vue
<script setup>
import { ref } from 'vue'

const activeTab = ref('details')
</script>

<template>
  <!-- Tabs toggle frequently -->
  <div v-show="activeTab === 'details'">Product details...</div>
  <div v-show="activeTab === 'reviews'">Customer reviews...</div>
  <div v-show="activeTab === 'specs'">Technical specs...</div>
</template>
```

## Important Caveats

### v-show doesn't work with `<template>`

```vue
<!-- This does NOT work! -->
<template v-show="condition">
  <p>Content</p>
</template>
```

Use `v-if` for `<template>` elements, or wrap in a `<div>`.

### v-if with v-for

**Never use `v-if` and `v-for` on the same element!** `v-if` has higher priority and won't have access to loop variables:

```vue
<!-- BAD - don't do this -->
<li v-for="user in users" v-if="user.isActive">...</li>

<!-- GOOD - filter in computed or use template -->
<template v-for="user in users" :key="user.id">
  <li v-if="user.isActive">{{ user.name }}</li>
</template>

<!-- BETTER - use computed property -->
<li v-for="user in activeUsers" :key="user.id">{{ user.name }}</li>
```

## Resources

- [Conditional Rendering](https://vuejs.org/guide/essentials/conditional.html) — Official Vue documentation on v-if, v-else, v-else-if, and v-show

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*