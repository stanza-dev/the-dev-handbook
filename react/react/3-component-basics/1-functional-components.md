---
source_course: "react"
source_lesson: "react-functional-components"
---

# Functional Components

## Introduction

Components are the building blocks of every React application. In modern React, components are plain JavaScript functions that accept props and return JSX. Understanding the rules and conventions of functional components is essential because every feature you build will be expressed as one or more components.

## Key Concepts

- **Function Component**: A JavaScript function whose name starts with a capital letter and returns JSX (or null).
- **Props**: The single object argument a component receives, containing all the data passed to it by its parent.
- **Pure Rendering**: A component should be a pure function of its props and state — given the same inputs, it must return the same output without side effects.
- **Component Tree**: Components compose into a tree where each parent renders children, forming the full UI hierarchy.

## Real World Context

Think of a design system at a company like Shopify. The team creates a `Button` component, a `Card` component, an `Avatar` component, and so on. Each is a self-contained function that accepts props (size, variant, onClick) and returns the appropriate markup. Developers across the company import and compose these components without needing to know their internal implementation. This modularity is what makes large applications maintainable.

## Deep Dive

A functional component is simply a function that returns JSX:

```tsx
function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}
```

### Component Rules

1. **Name must start with a capital letter.** React uses this convention to distinguish components from HTML elements. `<button>` renders a native HTML button; `<Button>` renders your custom component.

2. **Must return renderable content.** This can be JSX, a string, a number, `null` (to render nothing), or an array of these.

3. **Must be pure during rendering.** Do not mutate external variables, make network requests, or set timers during the render phase. Use hooks like `useEffect` for side effects.

```tsx
// A component with typed props and a default value
type AlertProps = {
  message: string;
  severity?: 'info' | 'warning' | 'error';
};

function Alert({ message, severity = 'info' }: AlertProps) {
  const colors = {
    info: 'blue',
    warning: 'orange',
    error: 'red',
  };

  return (
    <div style={{ borderLeft: `4px solid ${colors[severity]}`, padding: '1rem' }}>
      <strong>{severity.toUpperCase()}</strong>: {message}
    </div>
  );
}
```

### Arrow Functions vs Function Declarations

Both styles are valid. Function declarations are hoisted; arrow functions are not:

```tsx
// Function declaration (hoisted)
function Header() {
  return <header>My App</header>;
}

// Arrow function (not hoisted, assigned to a const)
const Footer = () => {
  return <footer>Copyright 2024</footer>;
};
```

Most teams pick one style and stay consistent. Function declarations are slightly more common for top-level components because they show up with a readable name in React DevTools by default.

## Common Pitfalls

1. **Lowercase component names** — Writing `function button()` and using `<button />` will render an HTML `<button>` element instead of your component. Always capitalize: `function Button()`.
2. **Side effects during render** — Calling `fetch()` or modifying a global variable inside the component body (outside of a hook) breaks React's purity model. Move side effects into `useEffect`.

## Best Practices

1. **One component per file** — Keep each component in its own file, named after the component (`Alert.tsx`). This makes imports predictable and code easier to find.
2. **Type your props** — Define a `Props` type or interface for every component. This catches bugs at compile time and serves as self-documentation.

## Summary

- Functional components are JavaScript functions that return JSX and accept a single props object.
- Component names must start with a capital letter and rendering must be free of side effects.
- Use TypeScript prop types for safety and arrow functions or declarations based on team preference.

## Code Examples

**A typed Button component with a variant prop and default value**

```tsx
type ButtonProps = {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
  onClick: () => void;
};

function Button({ children, variant = 'primary', onClick }: ButtonProps) {
  return (
    <button
      className={`btn btn--${variant}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// Usage
function App() {
  return (
    <div>
      <Button onClick={() => console.log('saved')} variant="primary">
        Save Changes
      </Button>
      <Button onClick={() => console.log('cancelled')} variant="secondary">
        Cancel
      </Button>
    </div>
  );
}
```


## Resources

- [Your First Component](https://react.dev/learn/your-first-component) — Official guide to creating your first React component
- [Keeping Components Pure](https://react.dev/learn/keeping-components-pure) — Official guide on component purity

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*