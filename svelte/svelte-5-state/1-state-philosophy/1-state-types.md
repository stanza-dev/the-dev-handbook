---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-state-types"
---

# Understanding Different Kinds of State

Before diving into implementation, let's understand the different types of state in a web application and where each belongs.

## 1. UI State (Local)

State that only matters to a single component:

```svelte
<script>
  // UI-only state - stays local
  let isDropdownOpen = $state(false);
  let selectedTab = $state('details');
  let inputValue = $state('');
</script>
```

**Examples:**
- Whether a dropdown/modal is open
- Current tab selection
- Form input values (before submission)
- Hover/focus states
- Animation states

## 2. Shared UI State

State that multiple components need to read or modify:

```svelte
<script>
  // Multiple components need this
  // Could be lifted to parent, or use Context/stores
  let notifications = $state([]);
  let sidebarCollapsed = $state(false);
</script>
```

**Examples:**
- Toast notifications
- Sidebar open/closed
- Theme (light/dark)
- Modal visibility controlled from multiple places

## 3. Server/Cache State

Data fetched from APIs that needs to be cached and synchronized:

```svelte
<script>
  // Server state - best handled by specialized libraries
  // Like TanStack Query, SWR, or SvelteKit's load functions
  let user = $state(null);
  let products = $state([]);
</script>
```

**Examples:**
- User profile data
- Product listings
- Comments, posts
- Any API response

## 4. URL State

State that lives in the URL and should be shareable:

```
/products?category=electronics&sort=price&page=2
```

**Examples:**
- Filters and search queries
- Pagination
- Selected items
- Feature flags

## The Golden Rule

**Start with the simplest solution that works:**

1. **Local state** (`$state` in component) - Start here
2. **Lifted state** (props from parent) - When siblings need to share
3. **Context** (`setContext`/`getContext`) - For deeply nested trees
4. **Global stores** (`.svelte.js` files) - For truly app-wide state
5. **External libraries** - For complex server state

📖 [Reactivity fundamentals](https://svelte.dev/docs/svelte/$state)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*