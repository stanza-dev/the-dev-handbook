---
source_course: "react-intermediate"
source_lesson: "react-starttransition-function"
---

# startTransition Function

## Introduction

Besides the `useTransition` hook, React exports a standalone `startTransition` function. Use it when you don't need the `isPending` state or can't use hooks.

## Key Concepts

**Import:**
```tsx
import { startTransition } from 'react';
```

**Usage:**
```tsx
startTransition(() => {
  setState(newValue);
});
```

## Real World Context

Useful in:
- Event handlers where you don't need loading indicators
- Utility functions outside components
- Class components (where hooks aren't available)
- Global state updates in stores

## Deep Dive

### When to Use Each

| Scenario | Use |
|----------|-----|
| Need `isPending` indicator | `useTransition` hook |
| Don't need pending state | `startTransition` function |
| Outside a component | `startTransition` function |
| In a class component | `startTransition` function |

### Use in Global State

```tsx
import { startTransition } from 'react';

// In a state management store
function setFilteredData(filter: string) {
  // The state update that subscribers see is a transition
  startTransition(() => {
    store.setState({ filter, data: computeFiltered(filter) });
  });
}
```

### Combining with External Libraries

When using third-party state libraries:

```tsx
import { startTransition } from 'react';
import { useStore } from 'zustand';

function SearchInput() {
  const setQuery = useStore(state => state.setQuery);
  
  function handleChange(e) {
    startTransition(() => {
      setQuery(e.target.value);
    });
  }
  
  return <input onChange={handleChange} />;
}
```

### Error Handling

Transitions don't swallow errors:

```tsx
startTransition(() => {
  // If this throws, the error propagates normally
  setData(riskyComputation());
});
```

## Common Pitfalls

1. **Assuming transitions track async work automatically**: While React 19 supports async callbacks in `startTransition`, the transition only tracks state updates within the callback. External async operations need their own loading state management.
2. **Missing visual feedback**: Without `useTransition`, there's no `isPending` - you need to track loading state another way if needed.
3. **Confusing transition scope with async completion**: In React 19, you can pass an async function to `startTransition`, but the transition tracks state updates inside it, not the full async duration.

## Best Practices

- Use `startTransition` for simple cases where you don't need `isPending`
- Use `useTransition` when you need to show loading indicators
- In React 19, async callbacks are supported in `startTransition`
- The transition tracks state updates within the callback
- Consider wrapping state library updates in transitions for better UX

## Summary

The `startTransition` function is a simpler alternative to `useTransition` when you don't need the `isPending` state. It works the same way - marking state updates as non-urgent transitions - but without the hook constraints.

## Code Examples

**startTransition function for simple transitions**

```tsx
import { startTransition, useState } from 'react';

// Simple case - no isPending needed
function QuickFilter({ items }: { items: Item[] }) {
  const [filter, setFilter] = useState('');
  const [filteredItems, setFilteredItems] = useState(items);

  function handleFilterChange(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    setFilter(value); // Urgent - update input immediately
    
    // No isPending needed, just defer the expensive work
    startTransition(() => {
      setFilteredItems(
        items.filter(item => 
          item.name.toLowerCase().includes(value.toLowerCase())
        )
      );
    });
  }

  return (
    <div>
      <input value={filter} onChange={handleFilterChange} />
      <List items={filteredItems} />
    </div>
  );
}

// Using with external state (e.g., URL state)
import { useRouter } from 'next/router';

function CategoryFilter() {
  const router = useRouter();

  function selectCategory(category: string) {
    // Navigation is expensive - mark as transition
    startTransition(() => {
      router.push(`/products?category=${category}`);
    });
  }

  return (
    <select onChange={(e) => selectCategory(e.target.value)}>
      <option value="all">All Categories</option>
      <option value="electronics">Electronics</option>
      <option value="clothing">Clothing</option>
    </select>
  );
}
```


## Resources

- [startTransition Reference](https://react.dev/reference/react/startTransition) — Official documentation for startTransition function

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*