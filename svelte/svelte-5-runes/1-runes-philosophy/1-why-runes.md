---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-why-runes"
---

# The Evolution of Svelte Reactivity

To understand why Runes exist, we need to understand what came before and why it wasn't enough.

## Svelte 4: Implicit Reactivity

In Svelte 4, reactivity was achieved through compiler analysis of assignments:

```svelte
<script>
  let count = 0;       // Svelte detects this declaration
  count += 1;          // Svelte sees this assignment → triggers update
  
  $: doubled = count * 2;  // Reactive statement
</script>
```

This felt magical — you just wrote normal JavaScript and things "just worked". But this magic had real problems:

### Problem 1: Component Boundaries

Reactivity ONLY worked inside `.svelte` files. You couldn't extract logic to a `.js` file:

```js
// utils.js — This does NOT work in Svelte 4!
let count = 0;  // Not reactive
export function increment() {
  count += 1;   // Doesn't trigger any updates
}
```

### Problem 2: The `$:` Confusion

The `$:` syntax was overloaded with two meanings:

```svelte
<script>
  $: doubled = count * 2;        // Reactive declaration
  $: console.log(count);         // Side effect
  $: if (count > 10) reset();    // Conditional side effect
</script>
```

When should you use `$:`? When does it re-run? The rules were complex and often surprising.

### Problem 3: Object/Array Reactivity

```svelte
<script>
  let items = [1, 2, 3];
  items.push(4);        // Does NOT trigger update!
  items = items;        // Workaround: reassign to trigger
</script>
```

## Runes: Explicit is Better Than Implicit

Svelte 5 introduces **Runes** — explicit symbols that clearly mark reactive behavior:

```svelte
<script>
  let count = $state(0);           // Explicitly reactive
  let doubled = $derived(count * 2); // Explicitly derived
  
  $effect(() => {                   // Explicit side effect
    console.log(count);
  });
</script>
```

No more guessing. Every reactive piece is clearly marked.

## The Signals Pattern

Runes implement the **Signals** pattern, used by Solid.js, Preact Signals, and Angular Signals. A signal is:

1. A container for a value
2. That tracks who reads it (subscribers)
3. And notifies subscribers when it changes

This enables **fine-grained updates** — when `count` changes, only the exact DOM nodes using `count` update, not the entire component.

## Resources

- [What Are Runes?](https://svelte.dev/docs/svelte/what-are-runes) — Official explanation of the Runes philosophy.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*