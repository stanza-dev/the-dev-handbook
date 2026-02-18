---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-headless-components"
---

# Separating Logic from Presentation

Headless components provide behavior without dictating UI. They're perfect for reusable logic.

## What is Headless?

A headless component:
- Manages state and logic
- Exposes that state to the consumer
- Lets the consumer decide how to render

## Example: Toggleable

```svelte
<!-- Toggleable.svelte -->
<script>
  let { 
    initial = false,
    children,
    ontoggle
  } = $props();
  
  let isOpen = $state(initial);
  
  function toggle() {
    isOpen = !isOpen;
    ontoggle?.(isOpen);
  }
  
  function open() {
    isOpen = true;
    ontoggle?.(true);
  }
  
  function close() {
    isOpen = false;
    ontoggle?.(false);
  }
</script>

{@render children(isOpen, { toggle, open, close })}
```

```svelte
<!-- Usage: Dropdown -->
<Toggleable>
  {#snippet children(isOpen, { toggle, close })}
    <div class="dropdown">
      <button onclick={toggle}>
        Menu {isOpen ? '▲' : '▼'}
      </button>
      {#if isOpen}
        <ul class="dropdown-menu">
          <li><a href="/profile">Profile</a></li>
          <li><a href="/settings">Settings</a></li>
          <li><button onclick={close}>Close</button></li>
        </ul>
      {/if}
    </div>
  {/snippet}
</Toggleable>

<!-- Usage: Accordion -->
<Toggleable>
  {#snippet children(isOpen, { toggle })}
    <div class="accordion">
      <button onclick={toggle}>
        FAQ Question {isOpen ? '−' : '+'}
      </button>
      {#if isOpen}
        <p>The answer to the question...</p>
      {/if}
    </div>
  {/snippet}
</Toggleable>
```

## Example: Async Data

```svelte
<!-- AsyncData.svelte -->
<script>
  let { url, children } = $props();
  
  let data = $state(null);
  let error = $state(null);
  let loading = $state(true);
  
  async function fetchData() {
    loading = true;
    error = null;
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error('Failed to fetch');
      data = await res.json();
    } catch (e) {
      error = e;
    } finally {
      loading = false;
    }
  }
  
  $effect(() => {
    fetchData();
  });
</script>

{@render children({ data, error, loading, refetch: fetchData })}
```

```svelte
<!-- Usage -->
<AsyncData url="/api/users">
  {#snippet children({ data, error, loading, refetch })}
    {#if loading}
      <Spinner />
    {:else if error}
      <ErrorMessage message={error.message} />
      <button onclick={refetch}>Retry</button>
    {:else}
      <UserList users={data} />
      <button onclick={refetch}>Refresh</button>
    {/if}
  {/snippet}
</AsyncData>
```

## Benefits

1. **Reusable Logic**: Same behavior, different presentations
2. **Testing**: Test logic separately from UI
3. **Flexibility**: Consumer controls rendering
4. **Type Safety**: Clear contract via snippet parameters

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*