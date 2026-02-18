---
source_course: "react"
source_lesson: "react-rendering-lists"
---

# Rendering Lists

Use JavaScript's `map()` to render arrays of data as lists of elements.

## The Key Prop

Always provide a unique `key` prop when rendering lists. Keys help React identify which items have changed, been added, or removed.

**Good keys:**
- Database IDs
- Unique identifiers from your data

**Bad keys:**
- Array indices (unless list is static)
- Random values generated on each render

## Code Examples

**Rendering a list with proper keys**

```tsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.completed ? '✅' : '⬜'} {todo.text}
        </li>
      ))}
    </ul>
  );
}

// Usage
const todos = [
  { id: 1, text: 'Learn React', completed: true },
  { id: 2, text: 'Build an app', completed: false },
];
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*