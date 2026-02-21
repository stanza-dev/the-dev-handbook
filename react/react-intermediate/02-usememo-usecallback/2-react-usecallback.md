---
source_course: "react-intermediate"
source_lesson: "react-usecallback"
---

# useCallback for Stable Functions

## Introduction

Every time a component re-renders, all functions declared inside it are recreated with new references. Usually this is harmless, but when you pass functions as props to memoized child components or list them as dependencies in effects, new references cause unnecessary re-renders or effect re-runs. The `useCallback` hook caches a function definition so it maintains the same reference across renders.

## Key Concepts

- **Function Identity**: In JavaScript, two identical function declarations are different objects. `(() => {}) !== (() => {})`.
- **useCallback**: Caches a function and returns the same reference as long as the dependencies have not changed.
- **React.memo**: A higher-order component that skips re-rendering when props have not changed by reference. `useCallback` ensures callback props maintain stable references.
- **Relationship to useMemo**: `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

## Real World Context

A data table component has 1,000 rows. Each row receives an `onDelete` callback. Without `useCallback`, every re-render of the parent creates a new `onDelete` function, which means every row re-renders even if the data has not changed. With `useCallback` and `React.memo`, only affected rows re-render.

## Deep Dive

### Basic Usage

```tsx
import { useCallback, memo, useState } from 'react';

const TodoItem = memo(function TodoItem({ todo, onToggle }: {
  todo: { id: number; text: string; done: boolean };
  onToggle: (id: number) => void;
}) {
  console.log('Rendering:', todo.text);
  return (
    <li onClick={() => onToggle(todo.id)}>
      {todo.done ? '\u2705' : '\u2B1C'} {todo.text}
    </li>
  );
});

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', done: false },
    { id: 2, text: 'Build an app', done: false },
  ]);
  const [filter, setFilter] = useState('');

  // Stable reference: TodoItem won't re-render when filter changes
  const handleToggle = useCallback((id: number) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  }, []);

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      <ul>
        {todos.map(todo => (
          <TodoItem key={todo.id} todo={todo} onToggle={handleToggle} />
        ))}
      </ul>
    </div>
  );
}
```

### useCallback for Effect Dependencies

```tsx
function SearchPage({ apiBase }: { apiBase: string }) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  // Stable function reference for the effect dependency
  const fetchResults = useCallback(async (searchQuery: string) => {
    const res = await fetch(`${apiBase}/search?q=${searchQuery}`);
    return res.json();
  }, [apiBase]);

  useEffect(() => {
    if (!query) return;
    fetchResults(query).then(setResults);
  }, [query, fetchResults]); // fetchResults only changes when apiBase changes

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

### useMemo vs useCallback

```tsx
// These are equivalent:
const memoizedFn = useCallback((x) => x * 2, []);
const memoizedFn = useMemo(() => (x) => x * 2, []);

// But they cache different things:
// useCallback caches the function itself
// useMemo caches the return value of the factory function
```

## Common Pitfalls

1. **Using useCallback without React.memo** — `useCallback` alone does nothing for performance. It only helps when the receiving component is wrapped in `React.memo` (or the function is used as an effect dependency). Without `memo`, the child re-renders regardless of prop reference stability.
2. **Adding too many dependencies** — If your callback depends on frequently changing values, `useCallback` will recreate the function on most renders anyway. Consider using functional state updates to reduce dependencies.

## Best Practices

1. **Pair useCallback with React.memo** — These two tools work together. Use `useCallback` on the callback and `React.memo` on the child that receives it.
2. **Prefer functional updates inside callbacks** — Using `setItems(prev => ...)` inside a `useCallback` means the callback does not need the current state in its dependency array, keeping it stable.

## Summary

- `useCallback` caches a function definition to maintain referential stability across re-renders.
- It prevents unnecessary re-renders when paired with `React.memo` on child components.
- Use functional state updates inside callbacks to minimize dependencies and maximize cache hits.

## Code Examples

**useCallback paired with React.memo to prevent unnecessary child re-renders**

```tsx
import { useCallback, memo } from 'react';

const ExpensiveChild = memo(function ExpensiveChild({
  onClick,
}: {
  onClick: () => void;
}) {
  console.log('ExpensiveChild rendered');
  return <button onClick={onClick}>Click me</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return (
    <div>
      <input value={name} onChange={e => setName(e.target.value)} />
      <p>Count: {count}</p>
      <ExpensiveChild onClick={handleClick} />
    </div>
  );
}
```


## Resources

- [useCallback Documentation](https://react.dev/reference/react/useCallback) — Official useCallback API reference

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*