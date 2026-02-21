---
source_course: "react-intermediate"
source_lesson: "react-memo-pitfalls"
---

# When memo Fails and Alternatives

## Introduction

React.memo is a powerful tool, but it is not a silver bullet. There are common situations where memo provides no benefit or even adds overhead. Understanding these limitations helps you choose the right optimization strategy.

## Key Concepts

- **Broken memoization**: Scenarios where memo cannot prevent re-renders because props change on every render despite having the same values.
- **Composition over memoization**: Restructuring components so that re-renders are naturally prevented without memo.
- **The children prop problem**: Passing JSX as children creates new React elements on every render, defeating memo.

## Real World Context

A developer wraps every component in memo hoping to improve performance, but notices no improvement. The React DevTools Profiler shows components still re-rendering because inline objects, functions, and JSX children create new references on every parent render, making the shallow comparison always return false.

## Deep Dive

**Problem 1: Inline objects and functions**

```tsx
// memo is useless here - new object and function every render
<MemoizedButton
  style={{ color: 'red' }}
  onClick={() => handleClick(id)}
/>

// Fixed with useMemo and useCallback
const style = useMemo(() => ({ color: 'red' }), []);
const handleClick = useCallback(() => onClick(id), [id, onClick]);
<MemoizedButton style={style} onClick={handleClick} />
```

**Problem 2: The children prop**

```tsx
// children is new JSX on every render - memo cannot help
<MemoizedCard>
  <p>This creates a new React element every render</p>
</MemoizedCard>
```

**Alternative: Composition (children as props pattern)**

Instead of memoizing, restructure so that the stateful part does not wrap the expensive part:

```tsx
// The children are created in App, not in ExpensiveWrapper
function App() {
  return (
    <StatefulWrapper>
      <ExpensiveChild /> {/* Stable reference - does not re-render */}
    </StatefulWrapper>
  );
}

function StatefulWrapper({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {children} {/* Same reference from parent */}
    </div>
  );
}
```

This works because the children JSX is created in App, which does not re-render when StatefulWrapper's internal state changes. The children reference stays stable without memo.

**Alternative: State colocation**

Move state closer to where it is used to reduce the blast radius of re-renders.

## Common Pitfalls

1. **Wrapping every component in memo** — This adds memory overhead for storing previous props and comparison overhead for every render, with no benefit if props always change.
2. **Ignoring composition patterns** — Often a component restructure is simpler and more effective than adding memo, useMemo, and useCallback everywhere.

## Best Practices

1. **Try composition first** — The children-as-props pattern and state colocation often solve re-render problems without any memoization.
2. **Measure the impact** — Use the React DevTools Profiler to verify that memo actually reduces render time. If it does not, remove it.

## Summary

- Memo fails when props change on every render (inline objects, functions, or children).
- Composition patterns like children-as-props can prevent re-renders without memoization.
- Always measure with the Profiler to confirm optimizations are effective.

## Code Examples

**Composition pattern as an alternative to memo**

```tsx
import { useState, ReactNode } from 'react';

// Composition: children don't re-render
function StatefulWrapper({ children }: { children: ReactNode }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      {children} {/* Stable reference */}
    </div>
  );
}

function App() {
  return (
    <StatefulWrapper>
      <ExpensiveChild /> {/* Never re-renders! */}
    </StatefulWrapper>
  );
}
```


## Resources

- [Before You memo()](https://overreacted.io/before-you-memo/) — Dan Abramov's article on composition patterns for performance

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*