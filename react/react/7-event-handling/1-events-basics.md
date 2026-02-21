---
source_course: "react"
source_lesson: "react-events-basics"
---

# Handling Events in React

## Introduction

Event handling is how your React application responds to user interactions — clicks, key presses, form submissions, mouse movements, and more. While the concept is similar to vanilla JavaScript event handling, React adds its own conventions and optimizations. Events use camelCase naming, handlers are functions (not strings), and React wraps native browser events in a cross-browser compatible SyntheticEvent system. Understanding these differences is essential for building interactive React applications.

## Key Concepts

- **camelCase naming**: React events use camelCase (`onClick`, `onChange`, `onSubmit`) instead of lowercase HTML attributes (`onclick`, `onchange`).
- **Function references**: Event handlers are JavaScript functions passed as references, not strings: `onClick={handleClick}` not `onclick="handleClick()"`.
- **SyntheticEvent**: React wraps the native browser event in a SyntheticEvent object that provides a consistent API across all browsers.
- **Prevent default**: Use `e.preventDefault()` instead of `return false` to prevent default browser behavior.

## Real World Context

Consider a shopping cart where users click "Add to Cart" buttons, hover over products for quick previews, press keyboard shortcuts for navigation, and submit checkout forms. Each of these interactions requires a different event handler. React's unified event system means you write the same patterns regardless of the event type, and the SyntheticEvent wrapper ensures consistent behavior across Chrome, Firefox, Safari, and Edge.

## Deep Dive

Basic event handling follows a simple pattern:

```tsx
function Button() {
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    console.log("Button clicked!");
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

Passing arguments to event handlers requires a wrapper function:

```tsx
function ItemList({ items }: { items: Item[] }) {
  const handleDelete = (id: string) => {
    console.log("Deleting item:", id);
  };

  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => handleDelete(item.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

Common event types and their TypeScript types:

```tsx
// Click events
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {};

// Form events
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};

// Input change events
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};

// Keyboard events
const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
  if (e.key === "Enter") {
    console.log("Enter pressed");
  }
};

// Focus events
const handleFocus = (e: React.FocusEvent<HTMLInputElement>) => {};
```

Event propagation works the same as in the DOM. Events bubble up from child to parent:

```tsx
function Parent() {
  return (
    <div onClick={() => console.log("Parent clicked")}>
      <button onClick={(e) => {
        e.stopPropagation(); // Prevents bubbling to parent
        console.log("Button clicked");
      }}>
        Click me
      </button>
    </div>
  );
}
```

## Common Pitfalls

1. **Calling the function instead of passing a reference** — Writing `onClick={handleClick()}` calls the function immediately during render. Use `onClick={handleClick}` to pass a reference, or `onClick={() => handleClick(arg)}` when you need to pass arguments.
2. **Forgetting `e.preventDefault()` on forms** — Without it, the browser performs a full page reload on form submission. Always call `e.preventDefault()` in `onSubmit` handlers (or use the new form `action` prop from React 19).

## Best Practices

1. **Name handlers with the `handle` prefix** — Use `handleClick`, `handleSubmit`, `handleChange` for handler functions and `onClick`, `onSubmit`, `onChange` for the props. This convention makes it clear what triggers what.
2. **Use TypeScript event types** — Always type your event parameters (`React.MouseEvent`, `React.ChangeEvent`, etc.) to get autocomplete and catch errors at build time.

## Summary

- React events use camelCase naming and accept function references, not strings.
- SyntheticEvent provides a consistent cross-browser API wrapping native DOM events.
- Use `e.preventDefault()` to prevent default behavior and `e.stopPropagation()` to stop event bubbling.

## Code Examples

**Common event handling patterns with TypeScript types**

```tsx
function EventDemo() {
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log("Clicked!", e.currentTarget.textContent);
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    console.log("Email:", formData.get("email"));
  };

  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === "Escape") e.currentTarget.blur();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        type="email"
        onChange={(e) => console.log(e.target.value)}
        onKeyDown={handleKeyDown}
      />
      <button onClick={handleClick}>Submit</button>
    </form>
  );
}
```


## Resources

- [Responding to Events](https://react.dev/learn/responding-to-events) — Official guide to event handling in React
- [SyntheticEvent Reference](https://react.dev/reference/react-dom/components/common#react-event-object) — React event object reference

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*