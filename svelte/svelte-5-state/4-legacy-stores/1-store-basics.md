---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-store-basics"
---

# Svelte Stores: The Pre-Runes Approach

Before Svelte 5's Runes, stores were THE way to share reactive state. While Runes are now preferred, you'll encounter stores in existing codebases and some libraries.

## What is a Store?

A store is an object with a `subscribe` method that allows reactive subscriptions:

```javascript
// The store contract
store = {
  subscribe(callback) {
    // Call callback with current value immediately
    // Call callback again whenever value changes
    // Return an unsubscribe function
  }
}
```

## writable: Read and Write

```javascript
import { writable } from 'svelte/store';

// Create a writable store with initial value
const count = writable(0);

// Read the value (subscription)
count.subscribe(value => {
  console.log('Count is:', value);
});

// Write a new value
count.set(5);  // Count is: 5

// Update based on current value
count.update(n => n + 1);  // Count is: 6
```

## readable: Read-Only

```javascript
import { readable } from 'svelte/store';

// Create a readable store
const time = readable(new Date(), (set) => {
  // This function runs when first subscriber appears
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);
  
  // Return cleanup function
  return () => clearInterval(interval);
});
```

## derived: Computed Values

```javascript
import { writable, derived } from 'svelte/store';

const count = writable(0);
const doubled = derived(count, $count => $count * 2);

// Multiple sources
const firstName = writable('John');
const lastName = writable('Doe');
const fullName = derived(
  [firstName, lastName],
  ([$first, $last]) => `${$first} ${$last}`
);
```

## Using Stores in Components

The `$` prefix auto-subscribes and unsubscribes:

```svelte
<script>
  import { count } from './stores.js';
</script>

<!-- $count auto-subscribes! -->
<p>Count: {$count}</p>

<button onclick={() => $count++}>+</button>
```

📖 [Stores documentation](https://svelte.dev/docs/svelte/stores)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*