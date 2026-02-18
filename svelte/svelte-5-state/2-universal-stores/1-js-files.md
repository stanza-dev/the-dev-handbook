---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-svelte-js-files"
---

# Universal Reactivity: Runes Outside Components

One of Svelte 5's most powerful features is that Runes work outside of `.svelte` components. You just need the right file extension.

## The Magic Extension

To use Runes in plain JavaScript files, use `.svelte.js` (or `.svelte.ts` for TypeScript):

```javascript
// counter.svelte.js
let count = $state(0);

export function increment() {
  count++;
}

export function decrement() {
  count--;
}

export function getCount() {
  return count;
}
```

## Why This Extension?

The Svelte compiler needs to know which files to process. Files ending in `.svelte.js` tell the compiler: "Hey, this file uses Runes - compile it!"

Regular `.js` files are not processed by Svelte, so `$state`, `$derived`, etc. won't work.

## Using the Store

```svelte
<!-- Counter.svelte -->
<script>
  import { increment, decrement, getCount } from './counter.svelte.js';
  
  // getCount() is reactive because it reads $state internally!
  let count = $derived(getCount());
</script>

<button onclick={decrement}>-</button>
<span>{count}</span>
<button onclick={increment}>+</button>
```

## Exporting Reactive Values Directly

You can also export reactive objects:

```javascript
// store.svelte.js
export const counter = $state({
  value: 0,
  increment() {
    this.value++;
  },
  decrement() {
    this.value--;
  }
});
```

```svelte
<!-- Component.svelte -->
<script>
  import { counter } from './store.svelte.js';
</script>

<button onclick={() => counter.increment()}>+</button>
<span>{counter.value}</span>
```

## File Extension Summary

| Extension | Svelte Compilation | Runes Work? |
|-----------|-------------------|-------------|
| `.js` | ❌ No | ❌ No |
| `.ts` | ❌ No | ❌ No |
| `.svelte` | ✅ Yes | ✅ Yes |
| `.svelte.js` | ✅ Yes | ✅ Yes |
| `.svelte.ts` | ✅ Yes | ✅ Yes |

📖 [Universal Reactivity documentation](https://svelte.dev/docs/svelte/$state#Reactivity-outside-components)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*