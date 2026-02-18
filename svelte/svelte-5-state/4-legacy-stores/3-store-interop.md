---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-store-interop"
---

# Stores Still Work in Svelte 5

Good news: your existing stores work perfectly in Svelte 5. The `$` prefix syntax is fully supported.

## Basic Usage Unchanged

```svelte
<script>
  import { count } from './stores.js'; // writable store
</script>

<!-- Works exactly like before -->
<p>Count: {$count}</p>
<button onclick={() => $count++}>+</button>
```

## Mixing Stores and Runes

You can use both in the same component:

```svelte
<script>
  import { externalStore } from 'some-library'; // Store from library
  
  // Your own rune-based state
  let localCount = $state(0);
  
  // Derived from store + local state
  let combined = $derived($externalStore + localCount);
</script>

<p>Store value: {$externalStore}</p>
<p>Local value: {localCount}</p>
<p>Combined: {combined}</p>
```

## Accessing Store Value Without $

If you need the actual store object (not the value):

```svelte
<script>
  import { count } from './stores.js';
  
  // Get current value imperatively
  let currentValue;
  const unsubscribe = count.subscribe(v => currentValue = v);
  unsubscribe(); // Don't forget to unsubscribe!
  
  // Or use get() helper
  import { get } from 'svelte/store';
  const snapshot = get(count); // Gets current value once
</script>
```

## Store Contract for Custom Objects

Any object with a proper `subscribe` method works:

```javascript
// This works as a store!
const myStore = {
  subscribe(callback) {
    callback('initial value');
    // Return unsubscribe function
    return () => {};
  }
};
```

```svelte
<script>
  import { myStore } from './myStore.js';
</script>

<p>{$myStore}</p> <!-- Works! -->
```

📖 [Stores documentation](https://svelte.dev/docs/svelte/stores)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*