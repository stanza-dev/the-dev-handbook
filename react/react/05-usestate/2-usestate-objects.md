---
source_course: "react"
source_lesson: "react-usestate-objects"
---

# State with Objects and Arrays

When state is an object or array, you must create a new reference to trigger re-renders.

## The Spread Operator

Always spread the previous state and update specific properties:

```jsx
setUser({ ...user, name: 'New Name' });
```

## Why?

React uses reference equality (`===`) to detect changes. Mutating the existing object won't trigger a re-render.

## Code Examples

**Managing array state immutably**

```tsx
function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', done: false }
  ]);

  const addTodo = (text) => {
    setTodos([
      ...todos,
      { id: Date.now(), text, done: false }
    ]);
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, done: !todo.done }
        : todo
    ));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (/* render todos */);
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*