---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-virtualization-concept"
---

# The Problem: Too Many DOM Nodes

Even with Svelte's efficiency, the DOM has limits.

## What Happens with 10,000 Items?

```svelte
<!-- ❌ This creates 10,000 DOM nodes -->
{#each items as item}
  <div class="item">
    <img src={item.image} />
    <h3>{item.title}</h3>
    <p>{item.description}</p>
  </div>
{/each}
```

**Problems:**
- 10,000 × 4 = 40,000+ DOM nodes
- Initial render takes seconds
- Scrolling stutters
- Memory usage explodes

## The Solution: Virtualization

Only render what's visible:

```
┌────────────────────────────┐
│     Items 1-100            │ ← Not rendered (above viewport)
│         ⋮                  │
├────────────────────────────┤
│ ┌────────────────────────┐ │
│ │ Visible: Items 101-120 │ │ ← RENDERED (in viewport)
│ └────────────────────────┘ │
├────────────────────────────┤
│         ⋮                  │
│     Items 121-10000        │ ← Not rendered (below viewport)
└────────────────────────────┘
```

Instead of 10,000 nodes, we render ~20-50 nodes!

## How Virtualization Works

1. Calculate which items are visible based on scroll position
2. Render only those items (plus a buffer)
3. Position them absolutely to maintain scroll height
4. Update on scroll

## Key Metrics

| Metric | No Virtualization | With Virtualization |
|--------|-------------------|--------------------|
| DOM Nodes | 10,000+ | ~50 |
| Initial Render | 2-5 seconds | <100ms |
| Memory | 100MB+ | ~10MB |
| Scroll Performance | Janky | Smooth 60fps |

📖 [Performance best practices](https://svelte.dev/docs/svelte/performance)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*