---
source_course: "react"
source_lesson: "react-functional-components"
---

# Functional Components

In modern React, components are JavaScript functions that return JSX.

## Rules

1. **Name must start with capital letter**: `Button`, not `button`
2. **Must return something**: JSX, `null`, or other renderable content
3. **Must be pure**: Same inputs → same output, no side effects during render

## Component Composition

Components can render other components, building a tree of UI elements.

## Code Examples

**Functional component with composition**

```tsx
// Simple component
function Button({ children, onClick }) {
  return (
    <button 
      className="btn" 
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// Composing components
function App() {
  return (
    <div>
      <Button onClick={() => alert('Clicked!')}>
        Click me
      </Button>
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*