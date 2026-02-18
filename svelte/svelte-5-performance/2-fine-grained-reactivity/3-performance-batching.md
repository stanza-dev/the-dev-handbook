---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-batching"
---

# How Svelte Batches Updates

Multiple state changes don't cause multiple DOM updates. Svelte batches them intelligently.

## The Problem Without Batching

```javascript
count = 1;  // DOM update?
count = 2;  // Another DOM update?
count = 3;  // Another DOM update?

// That would be 3 DOM updates for what should be 1!
```

## Svelte's Solution: Microtask Batching

```javascript
count = 1;  // Schedule update
count = 2;  // Already scheduled, just update value
count = 3;  // Already scheduled, just update value

// Later, in a microtask:
// → Single DOM update with count = 3
```

## Synchronous Updates Within Event Handlers

```svelte
<script>
  let count = $state(0);
  
  function handleClick() {
    count = 1;
    count = 2;
    count = 3;
    // DOM is NOT updated yet!
    
    // After this function returns, DOM updates once with count = 3
  }
</script>
```

## When You Need Immediate Updates

Rarely, you need to force DOM updates to happen immediately:

```svelte
<script>
  import { tick } from 'svelte';
  
  let count = $state(0);
  
  async function handleClick() {
    count = 5;
    
    // Wait for DOM to update
    await tick();
    
    // Now DOM is definitely updated
    console.log(document.querySelector('p').textContent); // "5"
  }
</script>

<p>{count}</p>
```

## Effect Timing

Effects also run in batched microtasks:

```svelte
<script>
  let a = $state(0);
  let b = $state(0);
  
  $effect(() => {
    console.log(a, b);  // Only logs ONCE per batch
  });
  
  function update() {
    a = 1;
    b = 2;
    // Effect runs once with (1, 2), not twice
  }
</script>
```

📖 [tick documentation](https://svelte.dev/docs/svelte/lifecycle-hooks#tick)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*