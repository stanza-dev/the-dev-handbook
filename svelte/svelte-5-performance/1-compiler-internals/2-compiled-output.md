---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-compiled-output"
---

# Examining Compiled Svelte Code

Let's see what Svelte actually generates. Understanding this helps you write more efficient code.

## Simple Component

**Input (.svelte):**
```svelte
<script>
  let count = $state(0);
</script>

<button onclick={() => count++}>
  Clicked {count} times
</button>
```

**What Svelte generates (conceptually):**
```javascript
// Simplified - actual output is more optimized
import { $state, template } from 'svelte/internal';

function create_fragment(ctx) {
  let button;
  let t0;
  let t1;
  
  return {
    create() {
      button = document.createElement('button');
      t0 = document.createTextNode('Clicked ');
      t1 = document.createTextNode(ctx.count);
      // ...
    },
    
    update(changed) {
      // ONLY updates what changed
      if (changed.count) {
        t1.data = ctx.count; // Direct DOM text update!
      }
    },
    
    destroy() {
      // Cleanup
    }
  };
}
```

## Key Observations

**1. Direct DOM creation**
```javascript
// Not virtual elements - real DOM nodes
button = document.createElement('button');
```

**2. Surgical updates**
```javascript
// Only touch what changed
t1.data = ctx.count;  // Just this text node
```

**3. No diffing needed**
The compiler already knows: "When `count` changes, update this specific text node."

## CSS Compilation

**Input:**
```svelte
<style>
  button { color: red; }
</style>
```

**Output:**
```css
/* Scoped with unique class */
button.svelte-abc123 { color: red; }
```

The button element gets `class="svelte-abc123"` added automatically.

📖 [Svelte compiler documentation](https://svelte.dev/docs/svelte/svelte-compiler)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*