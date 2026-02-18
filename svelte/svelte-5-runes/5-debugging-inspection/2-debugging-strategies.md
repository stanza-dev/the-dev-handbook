---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-debugging-strategies"
---

# Effective Debugging Techniques

Let's explore strategies for debugging common reactive issues.

## Problem 1: "Why isn't this updating?"

### Checklist:

1. **Is the variable wrapped in $state?**
```js
let count = 0;        // ❌ Not reactive
let count = $state(0); // ✅ Reactive
```

2. **For derived values, is it $derived?**
```js
let doubled = count * 2;          // ❌ Calculated once
let doubled = $derived(count * 2); // ✅ Reactive
```

3. **Are you in a .svelte or .svelte.js file?**
```
utils.js       // ❌ Runes don't work
utils.svelte.js // ✅ Runes work
```

4. **Is the dependency read synchronously?**
```js
$effect(() => {
  setTimeout(() => {
    console.log(count); // ❌ Not tracked (async)
  }, 100);
});
```

## Problem 2: "Infinite loop"

### Common Cause:
```js
$effect(() => {
  count = count + 1; // ❌ Reads and writes same variable
});
```

### Solution:
```js
// Use untrack() if you need to read without tracking
import { untrack } from 'svelte';

$effect(() => {
  const current = untrack(() => count); // Read without tracking
  // Do something with current
});
```

## Problem 3: "Stale closure"

### Issue:
```js
$effect(() => {
  const handler = () => {
    console.log(count); // Might be stale!
  };
  
  window.addEventListener('click', handler);
  return () => window.removeEventListener('click', handler);
});
```

### Solution:
```js
$effect(() => {
  const currentCount = count; // Capture current value
  
  const handler = () => {
    console.log(currentCount); // Now always current
  };
  
  // ...
});
```

## Using Browser DevTools

### Console Techniques:

```js
// Log with context
$inspect({ count, user, items });

// Group related logs
console.group('Cart Update');
console.log('Items:', items);
console.log('Total:', total);
console.groupEnd();

// Trace where updates come from
$inspect(count).with((type, value) => {
  console.trace(`count ${type}:`, value);
});
```

### Breakpoints:

```js
$inspect(count).with((type, value) => {
  if (type === 'update' && value > 100) {
    debugger; // Pause execution here
  }
});
```

## Svelte DevTools Extension

Install the Svelte DevTools browser extension to:
- Inspect component hierarchy
- View component props and state
- Track state changes visually
- Profile performance

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*