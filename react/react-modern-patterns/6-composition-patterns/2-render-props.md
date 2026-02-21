---
source_course: "react-modern-patterns"
source_lesson: "react-render-props"
---

# Render Props Pattern

## Introduction

The render props pattern is a technique for sharing behavior between components using a function prop. A component with a render prop receives a function that returns React elements, allowing it to control what is rendered based on its internal state or logic.

## Key Concepts

- **Render prop**: A function prop that a component calls to determine what to render. It passes its internal state or computed values to the function.
- **Headless component**: A component that provides behavior and state but no UI, delegating rendering entirely to the consumer.
- **Children as a function**: A variant where the render function is passed as the children prop instead of a named prop.

## Real World Context

A tooltip library needs to track hover state and mouse position but cannot know what the tooltip content looks like for every use case. Using a render prop, the library provides position data and hover state, while the consumer decides the visual presentation. Libraries like Downshift and React Virtuoso use this pattern extensively.

## Deep Dive

The classic render props pattern passes internal state to a function prop:

```tsx
type MousePosition = { x: number; y: number };

function MouseTracker({ render }: { render: (pos: MousePosition) => ReactNode }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e: MouseEvent) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return <>{render(position)}</>;
}

// Consumer controls the UI
<MouseTracker render={({ x, y }) => <p>Mouse: {x}, {y}</p>} />
```

**Modern context**: Custom hooks have largely replaced render props for sharing stateful logic. However, render props remain valuable for component-level composition where you need to pass behavior alongside a rendering boundary, such as virtualized list renderers or animation controllers.

The children-as-a-function variant is common in data provider patterns:

```tsx
<DataFetcher url="/api/users">
  {(data, loading) => loading ? <Spinner /> : <UserList users={data} />}
</DataFetcher>
```

## Common Pitfalls

1. **Using render props when a custom hook suffices** — If you only need shared logic without a rendering boundary, a custom hook is simpler and more idiomatic in modern React.
2. **Creating inline render functions** — Passing a new function on every render can cause unnecessary re-renders in children. Consider extracting the function if performance matters.

## Best Practices

1. **Prefer hooks for logic sharing** — Use custom hooks when you only need to share behavior. Reserve render props for when you need to share a rendering boundary.
2. **Use children-as-a-function for cleaner JSX** — When there is only one render prop, using children makes the JSX more readable.

## Summary

- Render props share behavior by passing internal state to a function that returns JSX.
- Custom hooks have replaced render props for most logic-sharing use cases.
- Render props remain useful for headless components and library APIs that need rendering boundaries.

## Code Examples

**Render props pattern for sharing mouse position behavior**

```tsx
import { useState, useEffect, ReactNode } from 'react';

type MousePosition = { x: number; y: number };

function MouseTracker({ render }: { render: (pos: MousePosition) => ReactNode }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  useEffect(() => {
    const handle = (e: MouseEvent) => setPos({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handle);
    return () => window.removeEventListener('mousemove', handle);
  }, []);
  return <>{render(pos)}</>;
}

// Usage
<MouseTracker render={({ x, y }) => <p>Position: {x}, {y}</p>} />
```


## Resources

- [Render Props](https://react.dev/learn/passing-props-to-a-component) — Official React guide on passing props to components

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*