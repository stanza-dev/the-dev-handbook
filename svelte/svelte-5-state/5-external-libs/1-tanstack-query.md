---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-tanstack-query"
---

# TanStack Query for Server State

For data that comes from APIs, specialized libraries like TanStack Query (formerly React Query) handle caching, refetching, and synchronization better than manual state management.

## Why Use TanStack Query?

**Without TanStack Query:**
```svelte
<script>
  let users = $state([]);
  let isLoading = $state(true);
  let error = $state(null);
  
  async function fetchUsers() {
    isLoading = true;
    try {
      const res = await fetch('/api/users');
      users = await res.json();
    } catch (e) {
      error = e;
    } finally {
      isLoading = false;
    }
  }
  
  $effect(() => {
    fetchUsers();
  });
</script>
```

**With TanStack Query:**
```svelte
<script>
  import { createQuery } from '@tanstack/svelte-query';
  
  const query = createQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(r => r.json())
  });
</script>

{#if $query.isLoading}
  <p>Loading...</p>
{:else if $query.error}
  <p>Error: {$query.error.message}</p>
{:else}
  {#each $query.data as user}
    <p>{user.name}</p>
  {/each}
{/if}
```

## What TanStack Query Gives You

1. **Automatic caching** - Data is cached and reused
2. **Background refetching** - Keeps data fresh
3. **Deduplication** - Multiple components requesting same data = one request
4. **Optimistic updates** - Update UI before server confirms
5. **Pagination/Infinite scroll** - Built-in support
6. **Devtools** - Visual debugging

## Setup in SvelteKit

```svelte
<!-- +layout.svelte -->
<script>
  import { QueryClient, QueryClientProvider } from '@tanstack/svelte-query';
  
  const queryClient = new QueryClient();
</script>

<QueryClientProvider client={queryClient}>
  <slot />
</QueryClientProvider>
```

## Mutations (Create/Update/Delete)

```svelte
<script>
  import { createMutation, useQueryClient } from '@tanstack/svelte-query';
  
  const queryClient = useQueryClient();
  
  const mutation = createMutation({
    mutationFn: (newUser) => 
      fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(newUser)
      }),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }
  });
</script>

<button onclick={() => $mutation.mutate({ name: 'New User' })}>
  Add User
</button>
```

📖 [TanStack Query Svelte docs](https://tanstack.com/query/latest/docs/svelte/overview)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*