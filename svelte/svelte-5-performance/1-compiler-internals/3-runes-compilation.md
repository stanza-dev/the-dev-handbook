---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-runes-compilation"
---

# Runes: Compile-Time Magic

Runes like `$state`, `$derived`, and `$effect` are not runtime functions - they're compile-time instructions that transform your code.

## $state Transformation

**What you write:**
```javascript
let count = $state(0);
count++;
```

**What the compiler creates (conceptually):**
```javascript
let count = {
  value: 0,
  subscribers: new Set(),
  
  get() {
    // Track who's reading
    if (currentEffect) {
      this.subscribers.add(currentEffect);
    }
    return this.value;
  },
  
  set(newValue) {
    this.value = newValue;
    // Notify all subscribers
    this.subscribers.forEach(effect => effect.run());
  }
};
```

Every read/write to `count` is transformed to use these getters/setters.

## $derived Transformation

**What you write:**
```javascript
let doubled = $derived(count * 2);
```

**What happens:**
1. Compiler wraps the expression in a reactive computation
2. When `count` changes, the computation re-runs
3. Only if the result differs, subscribers are notified

## $effect Transformation

**What you write:**
```javascript
$effect(() => {
  console.log(count);
});
```

**What the compiler creates:**
1. Wraps your function as a "reactive scope"
2. Tracks which reactive values are read during execution
3. Re-runs the function when any dependency changes
4. Handles cleanup when dependencies change or component unmounts

## Why "Runes" and Not Functions?

Runes look like function calls, but they're actually compiler macros:

```javascript
// This doesn't work - $state isn't a real function!
const createState = $state; // ❌ Error

// Runes must be used directly
let count = $state(0); // ✅ Compiler transforms this
```

The compiler sees `$state(0)` and transforms the entire variable declaration.

📖 [Runes documentation](https://svelte.dev/docs/svelte/what-are-runes)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*