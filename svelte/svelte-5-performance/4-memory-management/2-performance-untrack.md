---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-untrack"
---

# $untrack: Opting Out of Reactivity

Sometimes you need to read reactive values without creating dependencies. That's what `untrack` is for.

## The Problem: Unwanted Re-runs

```svelte
<script>
  let count = $state(0);
  let logCount = $state(0);
  
  // ❌ This runs whenever count OR logCount changes!
  $effect(() => {
    console.log(`Count changed to ${count}`);
    logCount++;  // Reading logCount creates a dependency
  });
</script>
```

This creates an infinite loop:
1. count changes → effect runs
2. effect increments logCount
3. logCount changed → effect runs
4. effect increments logCount...

## The Solution: untrack

```svelte
<script>
  import { untrack } from 'svelte';
  
  let count = $state(0);
  let logCount = $state(0);
  
  // ✅ Only re-runs when count changes
  $effect(() => {
    console.log(`Count changed to ${count}`);
    
    // Read logCount without tracking it
    untrack(() => {
      logCount++;
    });
  });
</script>
```

## When to Use untrack

**Use for:**
- Logging/analytics that shouldn't trigger re-runs
- Reading values for comparison without subscribing
- Breaking potential infinite loops

**Don't overuse:**
- If you're using `untrack` everywhere, reconsider your data flow

## Example: Analytics

```svelte
<script>
  import { untrack } from 'svelte';
  
  let searchTerm = $state('');
  let searchHistory = $state([]);
  
  $effect(() => {
    if (searchTerm.length > 2) {
      // We want to track searchTerm
      // but NOT searchHistory (would cause re-run when we push)
      untrack(() => {
        searchHistory = [...searchHistory, searchTerm];
      });
    }
  });
</script>
```

📖 [untrack documentation](https://svelte.dev/docs/svelte/$effect#$effect.tracking)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*