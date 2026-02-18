---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-global-singleton-warning"
---

# ⚠️ When Global Stores Go Wrong

Global stores in `.svelte.js` files are simple and powerful, but they have a critical limitation you must understand.

## The Problem: Server-Side Rendering (SSR)

In SvelteKit with SSR, modules are shared between ALL requests on the server:

```javascript
// store.svelte.js
// 🚨 DANGER: This is shared between ALL users on the server!
export const user = $state({
  name: '',
  isAuthenticated: false
});
```

**What happens:**
1. User A logs in → `user.name = 'Alice'`
2. User B makes a request
3. User B sees `user.name = 'Alice'` (User A's data!)

This is a **serious security vulnerability** and a source of bugs.

## When It's Safe to Use Global Stores

**Safe:**
- Client-only applications (no SSR)
- State that's truly global and doesn't vary per user
- Read-only configuration values

```javascript
// config.svelte.js - Safe: same for everyone
export const config = $state({
  apiUrl: 'https://api.example.com',
  theme: 'light'
});
```

**Unsafe:**
- User-specific data
- Shopping carts
- Form state
- Any data that varies per request

## The Solution: Context API

For SSR-safe state, use Context (covered in next section):

```svelte
<!-- +layout.svelte -->
<script>
  import { setContext } from 'svelte';
  import { createUserStore } from './user.svelte.js';
  
  // Each request gets its own instance!
  setContext('user', createUserStore());
</script>
```

## Quick Reference: When to Use What

| Scenario | Use |
|----------|-----|
| Client-only, truly global | Global `.svelte.js` stores |
| SSR app, user-specific | Context API |
| SSR app, per-request | Context + store factory |
| Server data caching | TanStack Query / SWR |

📖 [Context documentation](https://svelte.dev/docs/svelte/context)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*