---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-universal-reactivity"
---

# Breaking Free from Components

The killer feature of Runes is **universal reactivity** — Runes work anywhere, not just in `.svelte` files.

## The `.svelte.js` Extension

To use Runes in JavaScript files, use the `.svelte.js` (or `.svelte.ts`) extension:

```js
// counter.svelte.js
let count = $state(0);

export function increment() {
  count += 1;
}

export function getCount() {
  return count;
}
```

Now import and use it anywhere:

```svelte
<!-- App.svelte -->
<script>
  import { increment, getCount } from './counter.svelte.js';
</script>

<button onclick={increment}>
  Count: {getCount()}
</button>
```

## Why This Matters

### 1. Reusable State Logic

```js
// useCounter.svelte.js
export function createCounter(initial = 0) {
  let count = $state(initial);
  
  return {
    get value() { return count; },
    increment: () => count++,
    decrement: () => count--,
    reset: () => count = initial
  };
}
```

### 2. Shared Global State

```js
// auth.svelte.js
export const user = $state(null);
export const isLoggedIn = $derived(user !== null);

export async function login(email, password) {
  const response = await fetch('/api/login', { ... });
  user = await response.json();
}

export function logout() {
  user = null;
}
```

### 3. Complex Business Logic

```js
// cart.svelte.js
let items = $state([]);

export const cart = {
  get items() { return items; },
  get total() { return items.reduce((sum, item) => sum + item.price * item.qty, 0); },
  get count() { return items.reduce((sum, item) => sum + item.qty, 0); },
  
  add(product) {
    const existing = items.find(i => i.id === product.id);
    if (existing) {
      existing.qty += 1;
    } else {
      items.push({ ...product, qty: 1 });
    }
  },
  
  remove(productId) {
    items = items.filter(i => i.id !== productId);
  }
};
```

## The Power of Getters

Notice the `get` keyword above. When you return reactive state from a function, use getters to maintain reactivity:

```js
// ❌ Wrong: loses reactivity
function createCounter() {
  let count = $state(0);
  return {
    count,           // Captured value, not reactive!
    increment: () => count++
  };
}

// ✅ Correct: maintains reactivity
function createCounter() {
  let count = $state(0);
  return {
    get count() { return count; },  // Getter preserves reactivity
    increment: () => count++
  };
}
```

## Resources

- [.svelte.js Files](https://svelte.dev/docs/svelte/svelte-js-files) — Guide to using Runes outside components.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*