---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-context-basics"
---

# The Context API: SSR-Safe State Sharing

Context provides a way to pass data through the component tree without prop drilling, and it's safe for server-side rendering.

## Basic Usage

**Setting context (in a parent):**

```svelte
<!-- Parent.svelte -->
<script>
  import { setContext } from 'svelte';
  
  // Set a value that descendants can access
  setContext('user', {
    name: 'Alice',
    role: 'admin'
  });
</script>

<slot />
```

**Getting context (in any descendant):**

```svelte
<!-- DeepChild.svelte -->
<script>
  import { getContext } from 'svelte';
  
  // Access the context value
  const user = getContext('user');
</script>

<p>Welcome, {user.name}!</p>
```

## How Context Scoping Works

Context is scoped to the component tree:

```
┌─────────────────────────────────────────┐
│ App                                     │
│ ┌─────────────────────────────────────┐ │
│ │ Layout (setContext)                 │ │
│ │ ┌───────────────┐ ┌───────────────┐ │ │
│ │ │ Sidebar       │ │ Main          │ │ │
│ │ │ ✅ has access │ │ ✅ has access │ │ │
│ │ │ ┌───────────┐ │ │ ┌───────────┐ │ │ │
│ │ │ │ MenuItem  │ │ │ │ Content   │ │ │ │
│ │ │ │ ✅ access │ │ │ │ ✅ access │ │ │ │
│ │ │ └───────────┘ │ │ └───────────┘ │ │ │
│ │ └───────────────┘ └───────────────┘ │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ Footer (sibling, NOT descendant)    │ │
│ │ ❌ NO access (not a child of Layout)│ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## Why Context is SSR-Safe

Unlike global stores, context is created fresh for each component tree:

- **Server:** Each request renders a new component tree → new context
- **Client:** Each app instance has its own context
- **No data leaking** between users/requests!

## Important Rules

1. **Call during component init:** `setContext` and `getContext` must be called during component initialization, not inside callbacks or effects.

```svelte
<script>
  import { setContext, getContext } from 'svelte';
  
  // ✅ Called during init
  setContext('key', value);
  const ctx = getContext('key');
  
  function handleClick() {
    // ❌ Cannot call here!
    // setContext('key', newValue); // Error!
  }
</script>
```

2. **Context is readonly:** Once set, you can't change the context reference (but you can mutate the object if it's reactive).

📖 [Context documentation](https://svelte.dev/docs/svelte/context)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*