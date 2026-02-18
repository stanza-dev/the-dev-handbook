---
source_course: "react-performance"
source_lesson: "react-performance-compiler-introduction"
---

# Introduction to React Compiler

React Compiler automatically optimizes your components.

## What Is React Compiler?

Previously known as "React Forget", the React Compiler:

1. **Analyzes** your components at build time
2. **Automatically adds** memoization (memo, useMemo, useCallback)
3. **Generates optimized code** without manual intervention

## The Problem It Solves

```jsx
// Without compiler, you need manual optimization
function TodoList({ todos, filter }) {
  // Manual useMemo
  const filteredTodos = useMemo(() => {
    return todos.filter(t => t.status === filter);
  }, [todos, filter]);
  
  // Manual useCallback
  const handleToggle = useCallback((id) => {
    // toggle logic
  }, []);
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        // Manual memo needed for TodoItem
        <MemoizedTodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
        />
      ))}
    </ul>
  );
}

const MemoizedTodoItem = memo(TodoItem);
```

## With React Compiler

```jsx
// Write simple, readable code
function TodoList({ todos, filter }) {
  const filteredTodos = todos.filter(t => t.status === filter);
  
  const handleToggle = (id) => {
    // toggle logic
  };
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
        />
      ))}
    </ul>
  );
}

// Compiler automatically optimizes!
```

## How It Works

1. **Static Analysis**: Compiler analyzes data flow
2. **Dependency Tracking**: Identifies what each value depends on
3. **Automatic Memoization**: Inserts useMemo/useCallback where beneficial
4. **Component Optimization**: Wraps components with memo equivalent

## Requirements

- React 19+
- Strict Mode
- Following Rules of React

## Rules of React (Enforced by Compiler)

```jsx
// ❌ Violates rules - mutation
function Component({ items }) {
  items.push(newItem); // Don't mutate props!
  return <List items={items} />;
}

// ✅ Correct - create new array
function Component({ items }) {
  const newItems = [...items, newItem];
  return <List items={newItems} />;
}

// ❌ Violates rules - side effect in render
function Component() {
  const data = fetch('/api/data'); // Don't fetch in render!
  return <div>{data}</div>;
}

// ✅ Correct - use useEffect
function Component() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch('/api/data').then(setData);
  }, []);
  return <div>{data}</div>;
}
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*