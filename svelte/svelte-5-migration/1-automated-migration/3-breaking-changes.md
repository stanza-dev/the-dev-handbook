---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-breaking-changes"
---

# Breaking Changes in Svelte 5

While Svelte 5 maintains good backward compatibility, some things have changed.

## Event Modifiers Removed

```svelte
<!-- ❌ No longer works -->
<form on:submit|preventDefault={handler}>

<!-- ✅ Handle manually -->
<form onsubmit={(e) => {
  e.preventDefault();
  handler(e);
}}>
```

## beforeUpdate and afterUpdate Deprecated

```svelte
<!-- Svelte 4 -->
<script>
  import { beforeUpdate, afterUpdate } from 'svelte';
  
  beforeUpdate(() => { /* ... */ });
  afterUpdate(() => { /* ... */ });
</script>

<!-- Svelte 5 -->
<script>
  $effect.pre(() => {
    // Runs before DOM updates (like beforeUpdate)
  });
  
  $effect(() => {
    // Runs after DOM updates (like afterUpdate)
  });
</script>
```

## $$props and $$restProps

```svelte
<!-- Svelte 4 -->
<script>
  // $$props contains all props
  // $$restProps is props not explicitly declared
</script>

<div {...$$restProps}>...</div>

<!-- Svelte 5 -->
<script>
  let { children, class: className, ...rest } = $props();
</script>

<div class={className} {...rest}>
  {@render children()}
</div>
```

## Component Events Changed

```svelte
<!-- Svelte 4: Parent -->
<Child on:message={handleMessage} />

<!-- Svelte 5: Parent -->
<Child onmessage={handleMessage} />
```

## Immutable Arrays with Each Blocks

In Svelte 5, mutating arrays directly may not trigger updates in some cases:

```javascript
// ❌ May not trigger update
items.push(newItem);

// ✅ Always triggers update
items = [...items, newItem];
```

📖 [Breaking changes](https://svelte.dev/docs/svelte/v5-migration-guide#Breaking-changes)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*