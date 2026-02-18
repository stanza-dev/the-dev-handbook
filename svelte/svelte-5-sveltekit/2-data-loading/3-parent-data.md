---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-parent-data"
---

# Accessing Parent Load Data

Child routes can access data from parent layouts using the `parent()` function.

## Layout Data Flow

```javascript
// src/routes/+layout.server.js
export async function load() {
  return {
    user: await getAuthenticatedUser(),
    settings: await getUserSettings()
  };
}
```

```javascript
// src/routes/dashboard/+page.server.js
export async function load({ parent }) {
  // Wait for parent layout's data
  const { user } = await parent();
  
  // Use it to fetch page-specific data
  const dashboardData = await getDashboardForUser(user.id);
  
  return { dashboard: dashboardData };
}
```

## In Components

All ancestor data is automatically merged:

```svelte
<!-- src/routes/dashboard/+page.svelte -->
<script>
  let { data } = $props();
  
  // Available from layout
  console.log(data.user);
  console.log(data.settings);
  
  // Available from this page's load
  console.log(data.dashboard);
</script>
```

## Depends and Invalidation

Control when load functions re-run:

```javascript
// +page.server.js
export async function load({ depends }) {
  // Declare a custom dependency
  depends('app:cart');
  
  const cart = await getCart();
  return { cart };
}
```

```svelte
<!-- Component anywhere -->
<script>
  import { invalidate } from '$app/navigation';
  
  async function refreshCart() {
    // This re-runs any load function that called depends('app:cart')
    await invalidate('app:cart');
  }
</script>
```

## Invalidating on Navigation

```javascript
import { invalidateAll } from '$app/navigation';

// Re-run ALL load functions
await invalidateAll();

// Re-run load functions that depend on a URL
await invalidate('/api/posts');

// Re-run with custom key
await invalidate('app:user');
```

📖 [Invalidation documentation](https://svelte.dev/docs/kit/load#Rerunning-load-functions)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*