---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-reactive-data"
---

# Making Load Data Reactive

In Svelte 5, data from load functions comes via `$props()`. Understanding how to work with it reactively is crucial.

## The Basic Pattern

```svelte
<script>
  let { data } = $props();
</script>

<h1>{data.post.title}</h1>
```

## The Navigation Gotcha

When users navigate between pages, the `data` prop changes. Any derived state must handle this:

```svelte
<script>
  let { data } = $props();
  
  // ❌ BAD: This won't update when navigating!
  let sortedPosts = [...data.posts].sort((a, b) => 
    a.title.localeCompare(b.title)
  );
  
  // ✅ GOOD: Use $derived for reactive computations
  let sortedPosts = $derived(
    [...data.posts].sort((a, b) => a.title.localeCompare(b.title))
  );
</script>
```

## Complex Derived State

```svelte
<script>
  let { data } = $props();
  
  // Filter and sort options
  let searchTerm = $state('');
  let sortBy = $state('date');
  
  // Derived from BOTH props and local state
  let filteredPosts = $derived.by(() => {
    let result = data.posts;
    
    // Filter
    if (searchTerm) {
      result = result.filter(post => 
        post.title.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }
    
    // Sort
    return result.sort((a, b) => {
      if (sortBy === 'date') return new Date(b.date) - new Date(a.date);
      if (sortBy === 'title') return a.title.localeCompare(b.title);
      return 0;
    });
  });
</script>

<input bind:value={searchTerm} placeholder="Search..." />
<select bind:value={sortBy}>
  <option value="date">By Date</option>
  <option value="title">By Title</option>
</select>

{#each filteredPosts as post}
  <article>{post.title}</article>
{/each}
```

## Combining with Layout Data

```javascript
// +layout.server.js
export async function load() {
  return { user: await getUser() };
}
```

```svelte
<!-- +page.svelte -->
<script>
  // Page gets both its own data AND layout data merged
  let { data } = $props();
  
  // data.user comes from layout
  // data.posts comes from page
</script>
```

📖 [Load documentation](https://svelte.dev/docs/kit/load)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*