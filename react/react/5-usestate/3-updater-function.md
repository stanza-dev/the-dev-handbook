---
source_course: "react"
source_lesson: "react-usestate-updater-function"
---

# Functional Updates

## Introduction

When a state update depends on the previous state value, you should use the functional form of the setter: `setState(prev => newValue)`. This pattern is essential for correctness when multiple updates happen in the same event or when closures capture stale state values. This lesson explains why and when to use functional updates.

## Key Concepts

- **Functional Updater**: Passing a function (instead of a value) to the state setter. React calls it with the current pending state and uses the returned value as the new state.
- **Batching**: React groups multiple state updates within the same event handler into a single re-render for performance.
- **Stale Closures**: When a callback captures an old state value because the function was created during a previous render.
- **Queued Updates**: Each functional updater is queued and applied in order, each receiving the result of the previous one.

## Real World Context

In a real-time voting app, multiple vote submissions might arrive in rapid succession. If each update uses the captured `count` variable (not the functional form), only the last update will stick because they all reference the same stale value. Using `setCount(prev => prev + 1)` ensures each vote is counted correctly, regardless of how many updates are batched together.

## Deep Dive

### The Problem with Direct Updates

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleTripleIncrement = () => {
    setCount(count + 1); // Uses count = 0
    setCount(count + 1); // Still uses count = 0
    setCount(count + 1); // Still uses count = 0
    // Result: count becomes 1 (not 3!)
  };

  return <button onClick={handleTripleIncrement}>{count}</button>;
}
```

All three `setCount` calls capture the same `count` value from the closure. React batches them and the last one wins: `0 + 1 = 1`.

### The Solution: Functional Updates

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleTripleIncrement = () => {
    setCount(prev => prev + 1); // 0 + 1 = 1
    setCount(prev => prev + 1); // 1 + 1 = 2
    setCount(prev => prev + 1); // 2 + 1 = 3
    // Result: count becomes 3
  };

  return <button onClick={handleTripleIncrement}>{count}</button>;
}
```

Each updater function receives the latest pending state as `prev`, so updates chain correctly.

### When to Use Functional Updates

Use the functional form whenever the new state depends on the old state:

```tsx
// Toggling a boolean
setIsOpen(prev => !prev);

// Adding to an array
setItems(prev => [...prev, newItem]);

// Updating an object field
setUser(prev => ({ ...prev, name: 'Updated' }));

// Incrementing a counter
setCount(prev => prev + 1);
```

### When You Do NOT Need Functional Updates

If the new state does not depend on the previous state, pass the value directly:

```tsx
setName('Alice');           // Replacing entirely
setIsVisible(true);          // Setting to a known value
setSelectedId(event.target.value); // Setting from an event
```

## Common Pitfalls

1. **Always using direct updates by habit** — `setCount(count + 1)` works in simple cases but breaks with multiple updates or stale closures. Default to functional updates when deriving from previous state.
2. **Mutating inside the updater** — `setItems(prev => { prev.push(item); return prev; })` mutates the original array. Always return a new reference: `setItems(prev => [...prev, item])`.

## Best Practices

1. **Default to functional updates for derived state** — Any time the new value depends on the old value, use the function form. This eliminates an entire class of bugs related to stale closures and batching.
2. **Name the parameter descriptively** — Use `prev`, `current`, or an abbreviated form like `c` for count. Avoid single-letter names that obscure intent in complex updaters.

## Summary

- Use `setState(prev => newValue)` whenever the new state depends on the previous state.
- Functional updates guarantee each queued update receives the latest pending state, not a stale closure value.
- Direct value updates are fine when the new state is independent of the previous state.

## Code Examples

**Functional update to safely increment quantity or add new item**

```tsx
function ShoppingCart() {
  const [items, setItems] = useState<Array<{ id: number; name: string; qty: number }>>([]);

  const addItem = (product: { id: number; name: string }) => {
    setItems(prev => {
      const existing = prev.find(item => item.id === product.id);
      if (existing) {
        return prev.map(item =>
          item.id === product.id
            ? { ...item, qty: item.qty + 1 }
            : item
        );
      }
      return [...prev, { ...product, qty: 1 }];
    });
  };

  return <div>{/* render cart */}</div>;
}
```


## Resources

- [Queueing a Series of State Updates](https://react.dev/learn/queueing-a-series-of-state-updates) — Official guide on how React batches and queues state updates

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*