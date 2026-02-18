---
source_course: "react"
source_lesson: "react-usememo"
---

# useMemo Hook

`useMemo` caches the result of a calculation between re-renders.

## When to Use

1. **Expensive calculations**: Filtering large lists, complex math
2. **Referential equality**: Objects/arrays passed to memoized children

## Syntax

```jsx
const memoizedValue = useMemo(
  () => computeExpensiveValue(a, b),
  [a, b]
);
```

## Code Examples

**Memoizing filtered list**

```tsx
import { useMemo } from 'react';

function TodoList({ todos, filter }) {
  // Only recalculate when todos or filter changes
  const visibleTodos = useMemo(() => {
    console.log('Filtering todos...');
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

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*