---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-what-are-runes"
---

# Understanding Runes in Svelte 5

Runes are special symbols that start with a `$` sign. They tell the Svelte compiler to treat certain code specially — usually to make it **reactive**.

## Why Did Svelte Introduce Runes?

In Svelte 4 and earlier, reactivity was "magical". You'd write:

```svelte
<script>
  let count = 0;  // Svelte 4: This is automatically reactive!
</script>
```

Svelte's compiler would analyze your code and make `count` reactive behind the scenes. While elegant, this approach had problems:

1. **Only worked in `.svelte` files** — You couldn't have reactive state in plain `.js` files
2. **Confusing behavior** — Sometimes things weren't reactive when you expected them to be
3. **Limited TypeScript support** — The compiler's magic didn't play well with types

## Runes: Explicit is Better

Svelte 5 replaces the magic with explicit **Runes**. Now you write:

```svelte
<script>
  let count = $state(0);  // Svelte 5: Explicitly reactive!
</script>
```

The `$state()` rune clearly marks this variable as reactive. There's no guessing.

## The Main Runes

| Rune | Purpose | Example |
|------|---------|--------|
| `$state` | Create reactive state | `let count = $state(0)` |
| `$derived` | Computed values | `let doubled = $derived(count * 2)` |
| `$effect` | Side effects | `$effect(() => console.log(count))` |
| `$props` | Component props | `let { name } = $props()` |
| `$bindable` | Two-way binding | `let { value = $bindable() } = $props()` |

## The Big Win: Universal Reactivity

Runes work **everywhere** — not just in `.svelte` files. You can create reactive state in a `.js` or `.ts` file (with `.svelte.js` extension):

```js
// counter.svelte.js
export const counter = $state({ count: 0 });

export function increment() {
  counter.count += 1;
}
```

This enables powerful patterns for sharing state across your application.

## Resources

- [What Are Runes?](https://svelte.dev/docs/svelte/what-are-runes) — Official introduction to Svelte 5's Runes system.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*