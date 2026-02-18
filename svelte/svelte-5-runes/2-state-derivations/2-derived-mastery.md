---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-derived-mastery"
---

# Computed Values: $derived Deep Dive

Derived state is computed from other state. Svelte caches it and only recalculates when dependencies change.

## Basic $derived

For simple expressions:

```js
let price = $state(100);
let quantity = $state(2);
let tax = $state(0.1);

let subtotal = $derived(price * quantity);
let total = $derived(subtotal * (1 + tax));
```

The dependency chain is automatically tracked:
- `subtotal` depends on `price` and `quantity`
- `total` depends on `subtotal` and `tax`

## $derived.by for Complex Logic

When you need statements (if, for, try/catch), use `$derived.by`:

```js
let items = $state([...]);
let filter = $state('all');

let filteredItems = $derived.by(() => {
  if (filter === 'all') return items;
  if (filter === 'active') return items.filter(i => !i.completed);
  if (filter === 'completed') return items.filter(i => i.completed);
  return items;
});
```

## Complex Example: Shopping Cart

```js
let cart = $state([]);
let couponCode = $state('');

let cartStats = $derived.by(() => {
  const subtotal = cart.reduce((sum, item) => {
    return sum + item.price * item.quantity;
  }, 0);
  
  let discount = 0;
  if (couponCode === 'SAVE20') {
    discount = subtotal * 0.2;
  } else if (couponCode === 'FLAT10') {
    discount = Math.min(10, subtotal);
  }
  
  const total = subtotal - discount;
  const itemCount = cart.reduce((sum, item) => sum + item.quantity, 0);
  
  return {
    subtotal,
    discount,
    total,
    itemCount,
    isEmpty: cart.length === 0
  };
});
```

## Performance: Memoization

`$derived` values are **memoized**. The calculation only runs when dependencies change:

```js
// Expensive calculation
let sorted = $derived.by(() => {
  console.log('Sorting...');  // Only logs when 'items' changes
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
});

// These don't trigger re-sort:
otherState += 1;  // Unrelated state
console.log(sorted);  // Just reading
console.log(sorted);  // Reading again — still cached
```

## Derived vs Effect: Which to Use?

| Need | Use |
|------|-----|
| Compute a value from state | `$derived` |
| Perform a side effect (API call, DOM, logging) | `$effect` |
| Update the DOM title | `$effect` |
| Calculate filtered list | `$derived` |
| Sync to localStorage | `$effect` |

## Resources

- [$derived Documentation](https://svelte.dev/docs/svelte/$derived) — Complete reference for derived state.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*