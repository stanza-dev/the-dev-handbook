---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-virtual-list-implementation"
---

# Building a Virtual List in Svelte 5

Let's build a basic virtual list from scratch to understand how it works.

## The Basic Structure

```svelte
<script>
  let { items, itemHeight = 50 } = $props();
  
  let scrollTop = $state(0);
  let containerHeight = $state(0);
  
  // Calculate which items are visible
  let visibleRange = $derived.by(() => {
    const buffer = 5; // Extra items above/below
    const start = Math.max(0, Math.floor(scrollTop / itemHeight) - buffer);
    const visible = Math.ceil(containerHeight / itemHeight) + buffer * 2;
    const end = Math.min(items.length, start + visible);
    return { start, end };
  });
  
  // Items to actually render
  let visibleItems = $derived(
    items.slice(visibleRange.start, visibleRange.end).map((item, i) => ({
      item,
      index: visibleRange.start + i,
      style: `position: absolute; top: ${(visibleRange.start + i) * itemHeight}px; height: ${itemHeight}px; width: 100%;`
    }))
  );
  
  // Total height for scroll
  let totalHeight = $derived(items.length * itemHeight);
</script>

<div 
  class="container"
  bind:clientHeight={containerHeight}
  onscroll={(e) => scrollTop = e.currentTarget.scrollTop}
>
  <div class="content" style="height: {totalHeight}px; position: relative;">
    {#each visibleItems as { item, index, style } (item.id)}
      <div {style}>
        <slot {item} {index} />
      </div>
    {/each}
  </div>
</div>

<style>
  .container {
    overflow-y: auto;
    height: 100%;
  }
</style>
```

## Usage

```svelte
<script>
  import VirtualList from './VirtualList.svelte';
  
  let items = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }));
</script>

<div style="height: 500px">
  <VirtualList {items} itemHeight={40}>
    {#snippet children({ item, index })}
      <div class="row">
        #{index}: {item.name}
      </div>
    {/snippet}
  </VirtualList>
</div>
```

## Libraries to Consider

For production, use battle-tested libraries:
- `svelte-virtual-list` (Svelte 5 compatible)
- `@tanstack/svelte-virtual`
- Custom implementation for specific needs

📖 [Each blocks documentation](https://svelte.dev/docs/svelte/each)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*