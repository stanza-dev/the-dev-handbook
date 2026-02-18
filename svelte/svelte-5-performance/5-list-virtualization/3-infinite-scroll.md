---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-infinite-scroll"
---

# Implementing Infinite Scroll

Load more items as the user scrolls - a common pattern for feeds and lists.

## Basic Implementation

```svelte
<script>
  let items = $state([]);
  let isLoading = $state(false);
  let hasMore = $state(true);
  let page = $state(0);
  
  async function loadMore() {
    if (isLoading || !hasMore) return;
    
    isLoading = true;
    try {
      const newItems = await fetchItems(page);
      items = [...items, ...newItems];
      page++;
      hasMore = newItems.length > 0;
    } finally {
      isLoading = false;
    }
  }
  
  // Intersection Observer for trigger element
  function observeElement(node) {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          loadMore();
        }
      },
      { rootMargin: '200px' } // Load before reaching bottom
    );
    
    observer.observe(node);
    return {
      destroy: () => observer.disconnect()
    };
  }
  
  // Initial load
  loadMore();
</script>

<div class="feed">
  {#each items as item (item.id)}
    <article>
      <h2>{item.title}</h2>
      <p>{item.content}</p>
    </article>
  {/each}
  
  {#if isLoading}
    <div class="loading">Loading...</div>
  {/if}
  
  {#if hasMore && !isLoading}
    <div use:observeElement class="trigger"></div>
  {/if}
  
  {#if !hasMore}
    <p class="end">No more items</p>
  {/if}
</div>
```

## Combining with Virtualization

For the best of both worlds:

```svelte
<script>
  import VirtualList from './VirtualList.svelte';
  
  let items = $state([]);
  let isLoading = $state(false);
  
  async function handleScroll(event) {
    const { scrollTop, scrollHeight, clientHeight } = event.currentTarget;
    
    // Near bottom - load more
    if (scrollHeight - scrollTop - clientHeight < 200 && !isLoading) {
      await loadMore();
    }
  }
</script>

<VirtualList {items} onscroll={handleScroll}>
  <!-- Virtual list with infinite scroll -->
</VirtualList>
```

## Performance Tips

1. **Use placeholder heights** - Prevents layout shifts
2. **Debounce scroll handlers** - Don't fire too frequently
3. **Cancel pending requests** - When user scrolls away
4. **Cache loaded data** - Don't refetch on scroll back

📖 [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*