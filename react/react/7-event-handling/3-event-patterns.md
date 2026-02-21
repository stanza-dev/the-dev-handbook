---
source_course: "react"
source_lesson: "react-event-patterns"
---

# Event Handling Patterns

## Introduction

Beyond the basics of attaching event handlers, React applications use several established patterns for managing events at scale. These patterns handle common scenarios like passing data to handlers, conditionally handling events, building event-driven communication between components, and managing complex keyboard interactions. Mastering these patterns will make your components more reusable and your event handling code more maintainable.

## Key Concepts

- **Event handler props**: Passing event handler functions as props to child components, following the `onXxx` naming convention.
- **Event delegation in lists**: Using a single handler on a parent element to manage events from many child elements.
- **Keyboard navigation**: Building accessible keyboard interactions with `onKeyDown` and focus management.
- **Composed handlers**: Creating higher-order functions that combine multiple event behaviors.

## Real World Context

Consider a data table component with sortable columns, selectable rows, editable cells, and keyboard navigation. Each cell might need click, double-click, key press, and focus handlers. Without patterns for organizing this complexity, the code quickly becomes unmanageable. Event handler props let parent components control behavior, delegation reduces the number of listeners, and composed handlers keep individual handler functions focused on one responsibility.

## Deep Dive

**Pattern 1: Event handler props on custom components**

```tsx
// The component exposes an onXxx prop
function SearchInput({ onSearch }: { onSearch: (query: string) => void }) {
  const [query, setQuery] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSearch(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <button type="submit">Search</button>
    </form>
  );
}

// Parent controls what happens
function App() {
  return <SearchInput onSearch={(q) => console.log("Searching:", q)} />;
}
```

**Pattern 2: Composed event handlers**

```tsx
function composeHandlers<E>(
  ...handlers: Array<((e: E) => void) | undefined>
) {
  return (e: E) => {
    handlers.forEach((handler) => handler?.(e));
  };
}

// Usage: run both the internal and external handlers
function Input({ onChange, ...props }: React.InputHTMLAttributes<HTMLInputElement>) {
  const internalChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    analytics.track("input_change", { value: e.target.value });
  };

  return (
    <input
      onChange={composeHandlers(internalChange, onChange)}
      {...props}
    />
  );
}
```

**Pattern 3: Keyboard navigation**

```tsx
function Menu({ items }: { items: string[] }) {
  const [activeIndex, setActiveIndex] = useState(0);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case "ArrowDown":
        e.preventDefault();
        setActiveIndex((i) => Math.min(i + 1, items.length - 1));
        break;
      case "ArrowUp":
        e.preventDefault();
        setActiveIndex((i) => Math.max(i - 1, 0));
        break;
      case "Enter":
        console.log("Selected:", items[activeIndex]);
        break;
    }
  };

  return (
    <ul role="listbox" tabIndex={0} onKeyDown={handleKeyDown}>
      {items.map((item, i) => (
        <li key={i} role="option" aria-selected={i === activeIndex}
            className={i === activeIndex ? "active" : ""}>
          {item}
        </li>
      ))}
    </ul>
  );
}
```

## Common Pitfalls

1. **Creating new arrow functions in render for lists** — Writing `onClick={() => handle(id)}` inside a large list creates a new function per item per render. For most cases this is fine, but in very large lists (1000+ items), consider event delegation on the parent.
2. **Forgetting `e.preventDefault()` on keyboard handlers** — Arrow keys and Space scroll the page by default. Call `e.preventDefault()` in keyboard handlers to prevent the browser's default scrolling behavior.

## Best Practices

1. **Follow the `onXxx` / `handleXxx` naming convention** — Name props `onSearch`, `onSelect`, `onDelete` and internal handlers `handleSearch`, `handleSelect`, `handleDelete`. This makes the data flow immediately clear.
2. **Add keyboard support for accessibility** — Any interactive element that uses `onClick` should also support `onKeyDown` for Enter and Space keys, ensuring keyboard users can interact with your UI.

## Summary

- Use `onXxx` prop naming for event handler props passed to child components, with `handleXxx` for internal handlers.
- Composed handlers let you combine multiple behaviors (analytics, validation, external callbacks) into a single event handler.
- Keyboard navigation patterns with `onKeyDown` are essential for building accessible interactive components.

## Code Examples

**Event handler prop pattern with keyboard accessibility**

```tsx
// Event handler prop pattern
function ToggleButton({ onToggle, active }: {
  onToggle: (newState: boolean) => void;
  active: boolean;
}) {
  return (
    <button
      onClick={() => onToggle(!active)}
      onKeyDown={(e) => {
        if (e.key === "Enter" || e.key === " ") {
          e.preventDefault();
          onToggle(!active);
        }
      }}
      role="switch"
      aria-checked={active}
    >
      {active ? "ON" : "OFF"}
    </button>
  );
}

function App() {
  const [darkMode, setDarkMode] = useState(false);
  return <ToggleButton active={darkMode} onToggle={setDarkMode} />;
}
```


## Resources

- [Responding to Events](https://react.dev/learn/responding-to-events) — Official guide covering event handling patterns

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*