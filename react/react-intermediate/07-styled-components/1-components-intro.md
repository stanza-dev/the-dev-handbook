---
source_course: "react-intermediate"
source_lesson: "react-styled-components-intro"
---

# Introduction to Styled Components

## Introduction

Styled Components is the most popular CSS-in-JS library for React, letting you write actual CSS syntax directly inside your JavaScript files using tagged template literals. Each styled component is a real React component with its own automatically generated, unique class name. This approach ties styles directly to the components they belong to, eliminating the disconnect between separate CSS files and the components they style.

## Key Concepts

- **Tagged template literals**: Styled Components uses the JavaScript tagged template literal syntax (`styled.div\`...\``) to define CSS as a template string that gets processed at runtime.
- **Component generation**: `styled.button\`...\`` creates a new React component that renders a `<button>` with the specified styles and a unique, auto-generated class name.
- **Dynamic styling with props**: You can use JavaScript expressions inside the template literal to change styles based on component props.
- **Extending styles**: Use `styled(ExistingComponent)\`...\`` to create a new component that inherits all styles from the original and adds or overrides additional styles.

## Real World Context

Consider a design system team building a Button component used across a large application. The button has multiple variants (primary, secondary, ghost), sizes (small, medium, large), and states (loading, disabled). With Styled Components, all of these variations live in one file alongside the component logic. Dynamic props control which styles apply, and TypeScript can enforce that only valid variant/size combinations are used.

## Deep Dive

Basic styled component:

```tsx
import styled from "styled-components";

const Button = styled.button`
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;

  background-color: ${(props) => (props.primary ? "#2563eb" : "#6b7280")};
  color: white;

  &:hover {
    opacity: 0.9;
    transform: translateY(-1px);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    transform: none;
  }
`;
```

Extending an existing component:

```tsx
const LargeButton = styled(Button)`
  padding: 15px 30px;
  font-size: 20px;
`;

const OutlineButton = styled(Button)`
  background-color: transparent;
  border: 2px solid #2563eb;
  color: #2563eb;

  &:hover {
    background-color: #2563eb;
    color: white;
  }
`;
```

Usage in components:

```tsx
function App() {
  return (
    <div>
      <Button>Default</Button>
      <Button primary>Primary</Button>
      <LargeButton primary>Large Primary</LargeButton>
      <OutlineButton>Outline</OutlineButton>
    </div>
  );
}
```

Styled Components generates unique class names at runtime:

```html
<button class="sc-bczRLJ hUfkKi">Default</button>
<button class="sc-bczRLJ jXyZwQ">Primary</button>
```

The `attrs` method lets you set default HTML attributes:

```tsx
const SubmitButton = styled(Button).attrs({
  type: "submit",
})`
  width: 100%;
  margin-top: 16px;
`;
```

## Common Pitfalls

1. **Defining styled components inside render** — Creating a styled component inside a React component function means a new component is created every render, causing unmount/remount and losing state. Always define styled components outside components.
2. **Runtime performance overhead** — Styled Components injects styles at runtime using JavaScript, which adds overhead compared to static CSS. For performance-critical applications, consider CSS Modules or Tailwind.

## Best Practices

1. **Define styled components outside your React components** — Place them at the top of the file or in a separate `styles.ts` file to prevent re-creation on every render.
2. **Use TypeScript props interface** — Type the props your styled component accepts so the compiler catches invalid prop usage: `styled.button<{ primary?: boolean }>\`...\``.

## Summary

- Styled Components creates real React components with scoped, auto-generated class names using tagged template literals.
- Dynamic styling uses JavaScript expressions inside the template literal to respond to props.
- Always define styled components outside your React component functions to avoid performance issues from re-creation.

## Code Examples

**Typed styled components with variants and extension**

```tsx
import styled from "styled-components";

// Typed props for variants
interface ButtonProps {
  variant?: "primary" | "secondary" | "danger";
  size?: "sm" | "md" | "lg";
}

const sizes = { sm: "8px 12px", md: "10px 20px", lg: "14px 28px" };
const colors = {
  primary: "#2563eb",
  secondary: "#6b7280",
  danger: "#dc2626",
};

const Button = styled.button<ButtonProps>`
  padding: ${(p) => sizes[p.size || "md"]};
  background-color: ${(p) => colors[p.variant || "primary"]};
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  &:hover { opacity: 0.9; }
`;

// Extension
const IconButton = styled(Button)`
  display: inline-flex;
  align-items: center;
  gap: 8px;
`;

function App() {
  return (
    <>
      <Button variant="primary" size="lg">Save</Button>
      <Button variant="danger" size="sm">Delete</Button>
      <IconButton>+ Add Item</IconButton>
    </>
  );
}
```


## Resources

- [Styled Components Documentation](https://styled-components.com/docs) — Official styled-components documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*