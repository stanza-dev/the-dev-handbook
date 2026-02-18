---
source_course: "react"
source_lesson: "react-events-basics"
---

# Event Handling in React

React events are named using camelCase, and you pass a function as the event handler.

## Key Differences from HTML

| HTML | React |
|------|-------|
| `onclick="handleClick()"` | `onClick={handleClick}` |
| String handler | Function reference |
| `return false` to prevent default | `e.preventDefault()` |

## Synthetic Events

React wraps native browser events in SyntheticEvent objects for cross-browser compatibility.

## Code Examples

**Event handling patterns in React**

```tsx
function Button() {
  const handleClick = (e: React.MouseEvent) => {
    e.preventDefault();
    console.log('Button clicked!');
  };

  const handleMouseEnter = () => {
    console.log('Mouse entered');
  };

  return (
    <button
      onClick={handleClick}
      onMouseEnter={handleMouseEnter}
    >
      Click me
    </button>
  );
}

// Passing arguments to event handlers
function ItemList({ items }) {
  const handleDelete = (id: number) => {
    console.log('Deleting item:', id);
  };

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => handleDelete(item.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```


## Resources

- [Responding to Events](https://react.dev/learn/responding-to-events) — Official guide to event handling

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*