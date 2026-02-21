---
source_course: "react-intermediate"
source_lesson: "react-memoization-when"
---

# When to Memoize

## Introduction

Memoization is a powerful optimization tool, but it is not free. Every call to `useMemo`, `useCallback`, or `React.memo` adds complexity, increases memory usage, and makes code harder to read. This lesson teaches you how to identify when memoization is genuinely needed and when it does more harm than good.

## Key Concepts

- **Premature Optimization**: Adding performance optimizations before measuring whether there is a problem. This adds complexity without measurable benefit.
- **React DevTools Profiler**: A tool that records render timing and helps you identify which components are slow.
- **React Compiler**: A new tool (React 19+) that can automatically memoize components, potentially removing the need for manual useMemo and useCallback.
- **Cost-Benefit Analysis**: Memoization pays off only when the recalculation or re-render it prevents is more expensive than the memoization overhead.

## Real World Context

A junior developer joins a team and notices that none of the components use `useMemo` or `useCallback`. They spend a week wrapping every function and computed value in memoization hooks, thinking they are improving performance. In reality, the application was already fast, and now the code is harder to read and maintain. The better approach: profile first, optimize the specific bottlenecks found, and leave everything else simple.

## Deep Dive

### When TO Memoize

**1. Expensive calculations on large data:**

```tsx
// Yes: sorting 10,000 items is genuinely expensive
const sorted = useMemo(() => {
  return [...largeArray].sort(compareFn);
}, [largeArray, compareFn]);
```

**2. Stabilizing props for memoized children:**

```tsx
// Yes: prevents ExpensiveList from re-rendering when unrelated state changes
const handleSelect = useCallback((id: string) => {
  setSelectedId(id);
}, []);

return <MemoizedExpensiveList items={items} onSelect={handleSelect} />;
```

**3. Expensive object creation used as effect dependencies:**

```tsx
// Yes: prevents the effect from re-running due to new object reference
const config = useMemo(() => ({
  endpoint: apiUrl,
  headers: { Authorization: token },
}), [apiUrl, token]);

useEffect(() => {
  initializeClient(config);
}, [config]);
```

### When NOT to Memoize

**1. Cheap calculations:**

```tsx
// No: string concatenation is trivially cheap
const fullName = firstName + ' ' + lastName;
// Worse: const fullName = useMemo(() => firstName + ' ' + lastName, [firstName, lastName]);
```

**2. Primitive values:**

```tsx
// No: primitives are compared by value, not reference
const isEven = count % 2 === 0;
```

**3. Components that re-render infrequently:**

```tsx
// No: if this component only renders a few times, memo adds overhead with no gain
function AppHeader({ title }: { title: string }) {
  return <h1>{title}</h1>;
}
```

### The React Compiler

React 19 introduces an opt-in compiler that automatically applies memoization where beneficial. When enabled, you may no longer need manual `useMemo` and `useCallback` calls — the compiler handles it:

```tsx
// With the React Compiler, this is automatically optimized
function Dashboard({ items }) {
  const sorted = [...items].sort(compareFn); // Compiler auto-memoizes
  const handleClick = (id) => setSelected(id); // Compiler auto-memoizes

  return <ItemList items={sorted} onClick={handleClick} />;
}
```

### Profiling with React DevTools

Before adding any memoization, measure with the Profiler:

1. Open React DevTools and switch to the Profiler tab.
2. Click Record, interact with your app, then click Stop.
3. Look for components with long render times or that re-render unnecessarily.
4. Add memoization only to the specific components identified as bottlenecks.

## Common Pitfalls

1. **Memoizing without React.memo** — Wrapping callbacks in `useCallback` but not wrapping the child component in `React.memo` means the child still re-renders on every parent render. The `useCallback` does nothing useful.
2. **Over-memoizing** — Wrapping every value and function in memoization hooks makes the code verbose and harder to maintain. The dependency arrays themselves become a source of bugs.

## Best Practices

1. **Measure first, optimize second** — Use React DevTools Profiler to identify real bottlenecks before adding memoization.
2. **Try the React Compiler** — If your project is on React 19, enable the compiler and let it handle memoization automatically. Manual hooks may become unnecessary.

## Summary

- Memoize only when profiling reveals a genuine performance bottleneck: expensive calculations, unstable references causing unnecessary child re-renders, or object dependencies in effects.
- Do not memoize cheap computations, primitive values, or infrequently rendered components.
- The React Compiler (React 19) can automatically apply memoization, reducing the need for manual `useMemo` and `useCallback`.

## Code Examples

**Complete optimization: memo on child, useCallback on handler, useMemo on filtered data**

```tsx
import { useMemo, useCallback, memo } from 'react';

// Step 1: Identify that UserList re-renders are slow via Profiler
const UserList = memo(function UserList({ users, onSelect }: {
  users: User[];
  onSelect: (id: string) => void;
}) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id} onClick={() => onSelect(user.id)}>
          {user.name}
        </li>
      ))}
    </ul>
  );
});

// Step 2: Stabilize the callback that is passed to the memoized list
function Dashboard({ users }: { users: User[] }) {
  const [selectedId, setSelectedId] = useState<string | null>(null);
  const [searchQuery, setSearchQuery] = useState('');

  const handleSelect = useCallback((id: string) => {
    setSelectedId(id);
  }, []);

  const filteredUsers = useMemo(() => {
    return users.filter(u =>
      u.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [users, searchQuery]);

  return (
    <div>
      <input value={searchQuery} onChange={e => setSearchQuery(e.target.value)} />
      <UserList users={filteredUsers} onSelect={handleSelect} />
    </div>
  );
}
```


## Resources

- [React Compiler](https://react.dev/learn/react-compiler) — Official guide on the React Compiler for automatic memoization

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*