---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-deep-state"
---

# Understanding $state's Deep Reactivity

When you wrap a value with `$state`, Svelte creates a **deeply reactive proxy**. This has important implications.

## Deep Proxies

```js
let user = $state({
  name: 'Alice',
  address: {
    city: 'New York',
    zip: '10001'
  },
  hobbies: ['reading', 'coding']
});
```

All of these trigger updates:

```js
user.name = 'Bob';              // ✅ Tracked
user.address.city = 'Boston';   // ✅ Tracked (nested!)
user.hobbies.push('gaming');    // ✅ Tracked (array mutation!)
user.hobbies[0] = 'writing';    // ✅ Tracked (index assignment!)
```

This is a HUGE improvement over Svelte 4, where only top-level reassignments were tracked.

## Fine-Grained Tracking

Svelte tracks **which specific properties** are accessed:

```svelte
<script>
  let user = $state({
    name: 'Alice',
    email: 'alice@example.com'
  });
</script>

<!-- This component only re-renders when user.name changes -->
<p>Hello, {user.name}!</p>

<!-- Changing user.email won't cause the above to update -->
```

## $state.raw: Opting Out of Deep Reactivity

Sometimes you don't want deep proxying — for large objects or third-party instances:

```js
// Large dataset: don't proxy every property
let bigData = $state.raw(hugeDataset);

// Third-party object: proxying might break it
let map = $state.raw(new google.maps.Map());
```

With `$state.raw`, only reassigning the variable triggers updates:

```js
bigData.items[0].value = 'new';  // ❌ NOT tracked
bigData = { ...bigData };         // ✅ Tracked (reassignment)
```

## $state.snapshot: Getting Plain Objects

Sometimes you need the raw, non-proxied value (e.g., for `JSON.stringify` or sending to an API):

```js
let user = $state({ name: 'Alice' });

// Get a plain object copy
const plain = $state.snapshot(user);
console.log(plain);  // { name: 'Alice' } — not a proxy

// Useful for:
await fetch('/api/user', {
  body: JSON.stringify($state.snapshot(user))
});
```

## When to Use What

| Scenario | Use |
|----------|-----|
| Most state | `$state()` |
| Large, rarely-changing data | `$state.raw()` |
| Third-party object instances | `$state.raw()` |
| Passing to external APIs | `$state.snapshot()` |

## Resources

- [$state Documentation](https://svelte.dev/docs/svelte/$state) — Complete reference for the $state rune.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*