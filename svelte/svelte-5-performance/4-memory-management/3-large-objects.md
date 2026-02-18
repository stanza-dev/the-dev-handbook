---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-large-objects"
---

# Reactivity with Large Objects

Svelte 5's `$state` creates deep proxies. With large objects, this can have performance implications.

## Deep Reactivity by Default

```javascript
let users = $state([
  { id: 1, name: 'Alice', profile: { bio: '...', settings: {...} } },
  { id: 2, name: 'Bob', profile: { bio: '...', settings: {...} } },
  // ... thousands more
]);

// Every nested property is reactive!
users[0].profile.settings.theme = 'dark'; // Triggers updates
```

## When Deep Reactivity Is Expensive

1. **Very large arrays** (10,000+ items)
2. **Deeply nested objects** (many levels)
3. **Frequent bulk updates** (replacing entire arrays)

## Solution 1: $state.raw for Non-Reactive Data

```javascript
import { $state } from 'svelte';

// For data you won't mutate deeply
let rawData = $state.raw(hugeDataset);

// To update, replace the entire value
rawData = newHugeDataset;
```

## Solution 2: Selective Reactivity

```javascript
let items = $state.raw([...thousandsOfItems]);
let selectedId = $state(null); // Only this is deeply reactive

let selectedItem = $derived(
  items.find(item => item.id === selectedId)
);
```

## Solution 3: Pagination/Windowing

```javascript
let allItems = $state.raw([...thousandsOfItems]);
let page = $state(0);
let pageSize = 50;

let visibleItems = $derived(
  allItems.slice(page * pageSize, (page + 1) * pageSize)
);
```

## Measuring Performance

```javascript
$effect(() => {
  const start = performance.now();
  
  // Your reactive computation
  const result = processLargeData(data);
  
  const duration = performance.now() - start;
  if (duration > 16) { // More than one frame
    console.warn(`Slow computation: ${duration}ms`);
  }
});
```

📖 [$state.raw documentation](https://svelte.dev/docs/svelte/$state#$state.raw)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*