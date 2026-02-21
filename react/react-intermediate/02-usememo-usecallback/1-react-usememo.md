---
source_course: "react-intermediate"
source_lesson: "react-usememo"
---

# useMemo for Expensive Calculations

## Introduction

By default, React re-runs your entire component function every time it re-renders. Most of the time this is fast enough, but when a component performs an expensive calculation — sorting a large list, computing statistics, or generating complex data structures — recalculating on every render wastes time. The `useMemo` hook caches the result of a calculation and only recomputes it when its dependencies change.

## Key Concepts

- **Memoization**: Caching the result of a function call so that subsequent calls with the same inputs return the cached result instead of recomputing.
- **Dependency Array**: Like `useEffect`, `useMemo` takes a dependency array. The cached value is reused as long as the dependencies have not changed.
- **Referential Stability**: Objects and arrays created during render get new references every time. `useMemo` can stabilize these references to prevent unnecessary re-renders in child components.
- **Cost of Memoization**: `useMemo` itself has overhead (storing the previous value, comparing dependencies). It is only worthwhile when the calculation is genuinely expensive.

## Real World Context

A data analytics dashboard displays a table of 10,000 rows that can be sorted by any column and filtered by a search query. Without memoization, every keystroke in the search field would re-sort the entire dataset (even if the sort column did not change). With `useMemo`, sorting only recalculates when the sort criteria change, not when the search query updates.

## Deep Dive

### Basic Usage

```tsx
import { useMemo, useState } from 'react';

function ExpenseReport({ transactions }: { transactions: Transaction[] }) {
  const [sortBy, setSortBy] = useState<'date' | 'amount'>('date');
  const [filter, setFilter] = useState('');

  // Only re-sorts when transactions or sortBy changes
  const sorted = useMemo(() => {
    console.log('Sorting...');
    return [...transactions].sort((a, b) => {
      if (sortBy === 'date') return b.date.getTime() - a.date.getTime();
      return b.amount - a.amount;
    });
  }, [transactions, sortBy]);

  // Only re-filters when sorted array or filter changes
  const filtered = useMemo(() => {
    return sorted.filter(t =>
      t.description.toLowerCase().includes(filter.toLowerCase())
    );
  }, [sorted, filter]);

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} placeholder="Search..." />
      <select value={sortBy} onChange={e => setSortBy(e.target.value as 'date' | 'amount')}>
        <option value="date">Date</option>
        <option value="amount">Amount</option>
      </select>
      <table>
        <tbody>
          {filtered.map(t => (
            <tr key={t.id}><td>{t.description}</td><td>{t.amount}</td></tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

### Stabilizing Object References

```tsx
function ChartWrapper({ data, theme }: { data: number[]; theme: string }) {
  // Without useMemo, chartConfig is a new object every render
  const chartConfig = useMemo(() => ({
    data,
    colors: theme === 'dark' ? ['#fff'] : ['#000'],
    animate: true,
  }), [data, theme]);

  return <ExpensiveChart config={chartConfig} />;
}
```

If `ExpensiveChart` is wrapped in `React.memo`, stabilizing the config reference prevents unnecessary re-renders.

### When NOT to Use useMemo

```tsx
// Unnecessary: simple calculation
const fullName = useMemo(() => first + ' ' + last, [first, last]);
// Just compute it directly:
const fullName = first + ' ' + last;

// Unnecessary: primitive values are already stable
const doubled = useMemo(() => count * 2, [count]);
// Just compute it:
const doubled = count * 2;
```

## Common Pitfalls

1. **Memoizing everything** — Adding `useMemo` to cheap calculations adds complexity without measurable benefit. The overhead of memoization (storing previous values, comparing dependencies) can exceed the cost of the calculation itself.
2. **Wrong dependencies** — If you omit a dependency, the memoized value becomes stale. If you include an unstable dependency (an object created during render), the memo recalculates every time, defeating its purpose.

## Best Practices

1. **Profile before optimizing** — Use React DevTools Profiler to identify which calculations are actually slow. Do not guess.
2. **Prefer restructuring over memoizing** — Sometimes moving state down or splitting components eliminates the need for memoization entirely.

## Summary

- `useMemo` caches the result of an expensive calculation and recomputes only when dependencies change.
- Use it for genuinely expensive operations (sorting large datasets, complex computations) and for stabilizing object references.
- Do not memoize cheap calculations — the overhead is not worth it.

## Code Examples

**Memoizing a filtered list that only recalculates when todos or filter change**

```tsx
import { useMemo } from 'react';

function TodoList({ todos, filter }: { todos: Todo[]; filter: string }) {
  const visibleTodos = useMemo(() => {
    return todos.filter(todo => {
      if (filter === 'active') return !todo.completed;
      if (filter === 'completed') return todo.completed;
      return true;
    });
  }, [todos, filter]);

  return (
    <ul>
      {visibleTodos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```


## Resources

- [useMemo Documentation](https://react.dev/reference/react/useMemo) — Official useMemo API reference

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*