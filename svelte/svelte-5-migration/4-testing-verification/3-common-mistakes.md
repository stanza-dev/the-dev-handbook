---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-common-mistakes"
---

# Pitfalls to Avoid

Learn from common mistakes during Svelte 5 migration.

## 1. Forgetting $state

```svelte
<!-- ❌ Not reactive in runes mode -->
<script>
  let count = 0; // Forgot $state!
  $effect(() => console.log(count)); // Triggers $effect usage = runes mode
</script>

<!-- ✅ Correct -->
<script>
  let count = $state(0);
  $effect(() => console.log(count));
</script>
```

## 2. Assigning to $derived

```svelte
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);
  
  // ❌ Error! Can't assign to derived
  function reset() {
    doubled = 0;
  }
  
  // ✅ Change the source
  function reset() {
    count = 0;
  }
</script>
```

## 3. Missing Optional Chaining for Events

```svelte
<script>
  let { onsubmit } = $props();
</script>

<!-- ❌ Crashes if onsubmit not provided -->
<button onclick={() => onsubmit()}>Submit</button>

<!-- ✅ Safe with optional chaining -->
<button onclick={() => onsubmit?.()}>Submit</button>

<!-- ✅ Or provide default -->
<script>
  let { onsubmit = () => {} } = $props();
</script>
```

## 4. Forgetting to Render Children

```svelte
<script>
  let { children } = $props();
</script>

<!-- ❌ Children is a snippet, not a value -->
<div>{children}</div>

<!-- ✅ Must render it -->
<div>{@render children()}</div>
```

## 5. Array Mutation Without Reassignment

```svelte
<script>
  let items = $state(['a', 'b']);
</script>

<!-- ❌ May not update (depends on context) -->
<button onclick={() => items.push('c')}>Add</button>

<!-- ✅ Reassignment always works -->
<button onclick={() => items = [...items, 'c']}>Add</button>
```

## 6. Effect Without Cleanup

```svelte
<script>
  // ❌ Memory leak - interval never cleared
  $effect(() => {
    setInterval(() => count++, 1000);
  });
  
  // ✅ Return cleanup function
  $effect(() => {
    const i = setInterval(() => count++, 1000);
    return () => clearInterval(i);
  });
</script>
```

📖 [Migration pitfalls](https://svelte.dev/docs/svelte/v5-migration-guide)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*