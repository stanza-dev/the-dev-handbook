---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-keyed-each"
---

# The Importance of Keys in Lists

When rendering lists, keys help Svelte track which items are which, enabling efficient updates.

## The Problem Without Keys

```svelte
<script>
  let items = $state(['A', 'B', 'C']);
  
  function addToStart() {
    items = ['New', ...items];
  }
</script>

<!-- Without keys: Svelte may reuse DOM nodes incorrectly -->
{#each items as item}
  <input value={item} />
{/each}
```

Without keys, adding "New" to the start might:
1. Update the first input from "A" to "New"
2. Update the second input from "B" to "A"
3. Update the third input from "C" to "B"
4. Create a new input with "C"

This breaks any state stored in those inputs!

## The Solution: Add Keys

```svelte
<script>
  let items = $state([
    { id: 1, value: 'A' },
    { id: 2, value: 'B' },
    { id: 3, value: 'C' }
  ]);
  
  function addToStart() {
    items = [{ id: Date.now(), value: 'New' }, ...items];
  }
</script>

<!-- With keys: Svelte tracks each item correctly -->
{#each items as item (item.id)}
  <input value={item.value} />
{/each}
```

Now Svelte knows:
- The item with id 1 is still the same item
- Just need to create ONE new DOM node and insert it

## When Keys Matter Most

1. **Items can be reordered**
2. **Items have internal state** (inputs, animations, etc.)
3. **Items are added/removed frequently**

## Key Requirements

```svelte
<!-- ✅ Unique, stable identifiers -->
{#each items as item (item.id)}

<!-- ❌ Array index - bad! Changes when array changes -->
{#each items as item, index (index)}

<!-- ❌ Random values - bad! Different every render -->
{#each items as item (Math.random())}
```

## Impact on Performance

Without keys: O(n) updates when list changes
With keys: O(k) updates where k = actual changes

📖 [Each blocks documentation](https://svelte.dev/docs/svelte/each)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*