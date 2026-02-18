---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-memoization"
---

# Caching Expensive Computations

Svelte's `$derived` automatically memoizes values. Use it to avoid unnecessary recalculations.

## The Problem: Repeated Expensive Work

```svelte
<script>
  let items = $state([...thousandsOfItems]);
  let filter = $state('');
  let sortBy = $state('name');
  
  // ❌ BAD: This runs on EVERY render
  function getFilteredAndSorted() {
    return items
      .filter(item => item.name.includes(filter))
      .sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
  }
</script>

<!-- Called every time anything in component changes -->
{#each getFilteredAndSorted() as item}
  <div>{item.name}</div>
{/each}
```

## The Solution: $derived

```svelte
<script>
  let items = $state([...thousandsOfItems]);
  let filter = $state('');
  let sortBy = $state('name');
  
  // ✅ GOOD: Only recalculates when dependencies change
  let processedItems = $derived.by(() => {
    console.log('Processing...'); // Only logs when items/filter/sortBy change
    return items
      .filter(item => item.name.includes(filter))
      .sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
  });
</script>

{#each processedItems as item}
  <div>{item.name}</div>
{/each}
```

## When to Use $derived

✅ **Use for:**
- Filtering/sorting large arrays
- Complex calculations
- Data transformations
- Expensive formatting operations

❌ **Don't overuse for:**
- Simple property access (`$derived(user.name)` - unnecessary)
- Side effects (use `$effect` instead)

## Chained Derivations

```svelte
<script>
  let users = $state([...]);
  let searchTerm = $state('');
  let showActive = $state(true);
  
  // Each step is cached independently
  let filteredBySearch = $derived(
    users.filter(u => u.name.includes(searchTerm))
  );
  
  let filteredByActive = $derived(
    showActive 
      ? filteredBySearch.filter(u => u.active) 
      : filteredBySearch
  );
  
  let sortedUsers = $derived(
    [...filteredByActive].sort((a, b) => a.name.localeCompare(b.name))
  );
</script>
```

If only `showActive` changes, `filteredBySearch` doesn't recalculate!

📖 [Derived documentation](https://svelte.dev/docs/svelte/$derived)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*