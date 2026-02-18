---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-signal-mental-model"
---

# How Signals Work Under the Hood

Understanding the signal mental model helps you use Runes effectively.

## The Core Concepts

### 1. Signals (State)

A signal is a container that holds a value and knows who's watching it:

```js
let count = $state(0);  // Creates a signal

// Internally, Svelte creates something like:
// {
//   value: 0,
//   subscribers: Set of things depending on this
// }
```

### 2. Computed Values (Derived)

A computed value is a signal that depends on other signals:

```js
let doubled = $derived(count * 2);

// Svelte tracks that 'doubled' depends on 'count'
// When 'count' changes, 'doubled' recalculates
```

### 3. Effects

Effects are functions that run when their dependencies change:

```js
$effect(() => {
  document.title = `Count: ${count}`;
});

// Svelte sees 'count' was read during execution
// Adds this effect to count's subscriber list
```

## The Dependency Graph

Svelte builds a **reactive graph** of dependencies:

```
count ($state)
  ├── doubled ($derived)
  │     └── quadrupled ($derived)
  ├── DOM text node showing {count}
  └── $effect that updates title
```

When `count` changes:
1. `doubled` recalculates
2. `quadrupled` recalculates (depends on `doubled`)
3. The DOM text node updates
4. The effect runs

## Synchronous Tracking

**Critical concept**: Svelte only tracks dependencies read **synchronously**:

```js
$effect(() => {
  console.log(count);  // ✅ Tracked - read synchronously
  
  setTimeout(() => {
    console.log(other);  // ❌ NOT tracked - read asynchronously
  }, 100);
});
```

## Why This Matters

Understanding the graph helps you:
- Know what will trigger updates
- Avoid unnecessary recalculations
- Debug unexpected behavior
- Structure your reactive code efficiently

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*