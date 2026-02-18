---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-stores-vs-runes"
---

# Migrating from Stores to Runes

Svelte 5 prefers Runes, but stores still work. Here's how to choose and migrate.

## Comparison

| Feature | Stores | Runes |
|---------|--------|-------|
| API | `subscribe`, `set`, `update` | Direct property access |
| Syntax | `$store` prefix | No special syntax |
| File location | Any `.js` file | `.svelte.js` files only |
| Learning curve | Higher | Lower |
| TypeScript | Good | Better |
| Library support | Wide | Growing |

## Migration: writable to $state

**Store way:**
```javascript
import { writable } from 'svelte/store';

export const count = writable(0);

// Usage in component
$count++;
```

**Rune way:**
```javascript
// count.svelte.js
export const count = $state({ value: 0 });

// Usage in component
count.value++;
```

## Migration: derived to $derived

**Store way:**
```javascript
import { writable, derived } from 'svelte/store';

const count = writable(0);
const doubled = derived(count, $c => $c * 2);
```

**Rune way:**
```javascript
// counter.svelte.js
let count = $state(0);
let doubled = $derived(count * 2);

export { count, doubled };
```

## When Stores Still Make Sense

1. **Existing codebase:** Don't rewrite working code just because
2. **Third-party libraries:** Many still use stores
3. **Standard `.js` files:** Runes require `.svelte.js`
4. **Complex subscriptions:** Custom store logic

## Custom Stores (Still Useful)

```javascript
import { writable } from 'svelte/store';

function createPersisted(key, initial) {
  const stored = localStorage.getItem(key);
  const value = stored ? JSON.parse(stored) : initial;
  
  const { subscribe, set, update } = writable(value);
  
  return {
    subscribe,
    set: (v) => {
      localStorage.setItem(key, JSON.stringify(v));
      set(v);
    },
    update: (fn) => {
      update(v => {
        const newValue = fn(v);
        localStorage.setItem(key, JSON.stringify(newValue));
        return newValue;
      });
    }
  };
}

export const preferences = createPersisted('prefs', { theme: 'light' });
```

📖 [Stores documentation](https://svelte.dev/docs/svelte/stores)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*