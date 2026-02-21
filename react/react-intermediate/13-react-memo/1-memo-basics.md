---
source_course: "react-intermediate"
source_lesson: "react-memo-basics"
---

# Memoizing Components with memo

## Introduction

When a parent component re-renders, all of its child components also re-render by default, even if their props have not changed. React.memo is a higher-order component that wraps your component and skips re-rendering when the props are the same as the previous render.

## Key Concepts

- **React.memo**: A higher-order component that memoizes your component, performing a shallow comparison of props to decide whether to re-render.
- **Shallow comparison**: Compares props by reference (`===`). Primitive values are compared by value; objects and arrays are compared by reference.
- **Referential equality**: Two objects are referentially equal only if they point to the same memory location.

## Real World Context

A product listing page renders 200 product cards. When the user adds an item to the cart (updating state in the parent), all 200 cards re-render even though only the cart changed. Wrapping ProductCard in React.memo means each card only re-renders when its own props actually change, saving potentially thousands of wasted render cycles.

## Deep Dive

Using memo is straightforward. Wrap your component function with `memo()`:

```tsx
import { memo, useState } from 'react';

const ExpensiveList = memo(function ExpensiveList({ items }: { items: Item[] }) {
  console.log('ExpensiveList rendered');
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

function App() {
  const [count, setCount] = useState(0);
  const [items] = useState([{ id: 1, name: 'Item 1' }]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      {/* Won't re-render when count changes */}
      <ExpensiveList items={items} />
    </div>
  );
}
```

The key insight is that memo uses shallow comparison. If you pass a new array or object reference on every render, memo cannot help because the references differ even if the contents are identical. This is why `useMemo` and `useCallback` are often used alongside `memo` to stabilize props.

Memo also accepts an optional custom comparison function as a second argument for more complex equality checks.

## Common Pitfalls

1. **Passing new objects or arrays as props** — Creating `{ style: { color: 'red' } }` inline defeats memo because it is a new object every render. Use `useMemo` to stabilize it.
2. **Memoizing everything** — Memo has a cost: it stores the previous props and performs a comparison. For simple, cheap-to-render components, the overhead may exceed the savings.

## Best Practices

1. **Combine with useMemo and useCallback** — Stabilize object, array, and function props passed to memoized components.
2. **Profile before memoizing** — Use React DevTools Profiler to confirm a component is actually re-rendering unnecessarily before adding memo.

## Summary

- React.memo skips re-rendering when props have not changed (shallow comparison).
- It is most effective for expensive components that receive stable props.
- Always profile first; memo has overhead and is not free.

## Code Examples

**Preventing re-renders with React.memo**

```tsx
import { memo, useState } from 'react';

const ExpensiveList = memo(function ExpensiveList({ items }: { items: Item[] }) {
  console.log('ExpensiveList rendered');
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

function App() {
  const [count, setCount] = useState(0);
  const [items] = useState([{ id: 1, name: 'Item 1' }]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveList items={items} />
    </div>
  );
}
```


## Resources

- [memo](https://react.dev/reference/react/memo) — Official React.memo documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*