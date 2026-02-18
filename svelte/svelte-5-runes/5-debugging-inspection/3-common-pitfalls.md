---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-common-pitfalls"
---

# Mistakes Everyone Makes (And How to Avoid Them)

## Pitfall 1: Forgetting $state

```js
// ❌ Beginner mistake
let count = 0;
count += 1; // Nothing updates!

// ✅ Correct
let count = $state(0);
count += 1; // Updates!
```

## Pitfall 2: Using $effect for Derived Values

```js
// ❌ Anti-pattern
let doubled;
$effect(() => {
  doubled = count * 2;
});

// ✅ Correct
let doubled = $derived(count * 2);
```

## Pitfall 3: Returning State Instead of Getters

```js
// ❌ Loses reactivity
function createStore() {
  let value = $state(0);
  return { value }; // 'value' is captured!
}

// ✅ Maintains reactivity
function createStore() {
  let value = $state(0);
  return { 
    get value() { return value; } 
  };
}
```

## Pitfall 4: Async in Effects

```js
// ❌ userId won't be tracked
$effect(() => {
  fetch('/api/user').then(() => {
    console.log(userId); // Read async — not tracked!
  });
});

// ✅ Capture sync, use async
$effect(() => {
  const id = userId; // Captured synchronously
  fetch(`/api/user/${id}`).then(...);
});
```

## Pitfall 5: Mutating Props

```js
// ❌ Don't mutate props directly
let { user } = $props();
user.name = 'New Name'; // Might work, but bad practice!

// ✅ Use $bindable or callbacks
let { user, onupdate } = $props();
onupdate({ ...user, name: 'New Name' });
```

## Pitfall 6: Forgetting .svelte.js Extension

```js
// utils.js — ❌ Runes are compiled away, no reactivity
export const count = $state(0);

// utils.svelte.js — ✅ Runes work
export const count = $state(0);
```

## Pitfall 7: Object Identity

```js
// ❌ Creates new array every time
let items = $derived([...rawItems].sort());

// Each render creates new identity, could cause issues
// if used as key or compared with ===
```

## Pitfall 8: Missing Cleanup

```js
// ❌ Memory leak
$effect(() => {
  window.addEventListener('scroll', handler);
});

// ✅ Clean up
$effect(() => {
  window.addEventListener('scroll', handler);
  return () => window.removeEventListener('scroll', handler);
});
```

## Quick Reference

| Want to... | Use |
|------------|-----|
| Create reactive variable | `$state()` |
| Compute from other state | `$derived()` |
| Run side effects | `$effect()` |
| Receive props | `$props()` |
| Enable two-way binding | `$bindable()` |
| Debug reactivity | `$inspect()` |

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*