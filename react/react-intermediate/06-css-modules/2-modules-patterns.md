---
source_course: "react-intermediate"
source_lesson: "react-css-modules-patterns"
---

# CSS Modules Patterns

## Introduction

Now that you understand the basics of CSS Modules, let us explore the patterns that make them powerful in production React applications. From handling dynamic class names and responsive design to integrating with CSS custom properties and building theme systems, these patterns will help you write maintainable, scalable styles.

## Key Concepts

- **Dynamic class names**: Building class name strings from props and state to apply conditional styles.
- **`classnames` utility**: A popular library (or simple helper function) for conditionally joining class names without messy string concatenation.
- **CSS custom properties (variables)**: Using CSS variables with CSS Modules to create themeable components without JavaScript overhead.
- **Global class escape hatch**: CSS Modules provide a `:global()` selector for when you need to target global class names or third-party library classes.

## Real World Context

A component library might have a Button component with six variants (primary, secondary, danger, ghost, outline, link), three sizes (small, medium, large), and states (loading, disabled). Managing all these combinations with conditional string concatenation creates unreadable code. The `classnames` utility and well-structured CSS Module patterns make this manageable and type-safe.

## Deep Dive

**Dynamic class names with a helper:**

```tsx
// Simple classnames helper (or use the 'classnames' npm package)
function cn(...classes: (string | boolean | undefined | null)[]) {
  return classes.filter(Boolean).join(" ");
}

import styles from "./Button.module.css";

function Button({ variant = "primary", size = "md", loading, children }: {
  variant?: "primary" | "secondary" | "danger";
  size?: "sm" | "md" | "lg";
  loading?: boolean;
  children: React.ReactNode;
}) {
  return (
    <button className={cn(
      styles.button,
      styles[variant],
      styles[size],
      loading && styles.loading,
    )}>
      {loading ? "Loading..." : children}
    </button>
  );
}
```

**CSS custom properties for theming:**

```css
/* Card.module.css */
.card {
  background-color: var(--card-bg, #ffffff);
  color: var(--card-text, #1f2937);
  border: 1px solid var(--card-border, #e5e7eb);
  border-radius: 8px;
  padding: 16px;
}
```

```tsx
import styles from "./Card.module.css";

function ThemedCard({ dark, children }: { dark?: boolean; children: React.ReactNode }) {
  const themeVars = dark ? {
    "--card-bg": "#1f2937",
    "--card-text": "#f9fafb",
    "--card-border": "#374151",
  } : {};

  return (
    <div className={styles.card} style={themeVars as React.CSSProperties}>
      {children}
    </div>
  );
}
```

**Global class escape hatch:**

```css
/* Override third-party library styles */
.wrapper :global(.third-party-class) {
  color: red;
}

/* Mix local and global */
.myComponent :global(.active) {
  font-weight: bold;
}
```

**Responsive patterns:**

```css
/* Grid.module.css */
.grid {
  display: grid;
  gap: 16px;
  grid-template-columns: 1fr;
}

@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

## Common Pitfalls

1. **Trying to use dynamic class names as strings** — Writing `styles[`variant-${name}`]` will fail if the class name does not exist in the CSS file. CSS Modules only export class names that actually exist in the stylesheet.
2. **Over-using `:global()`** — The global escape hatch defeats the purpose of CSS Modules. Use it sparingly for third-party library overrides, not for your own styles.

## Best Practices

1. **Use a `cn` helper for conditional classes** — Whether you use the `classnames` library or a simple helper, avoid manual string concatenation. It produces cleaner, more readable code.
2. **Prefer CSS custom properties over prop-based inline styles** — For theming and dynamic values, CSS variables keep the styling in CSS where it belongs and perform better than inline style objects.

## Summary

- Use a `classnames` helper function or library to cleanly combine conditional CSS Module classes.
- CSS custom properties let you build themeable components within CSS Modules without JavaScript-based style objects.
- The `:global()` selector provides an escape hatch for targeting global or third-party classes when necessary.

## Code Examples

**Dynamic variant classes with cn helper and CSS Modules**

```tsx
// cn helper for conditional classes
function cn(...classes: (string | boolean | undefined | null)[]) {
  return classes.filter(Boolean).join(" ");
}

import styles from "./Alert.module.css";

type AlertProps = {
  variant: "info" | "success" | "warning" | "error";
  dismissible?: boolean;
  children: React.ReactNode;
};

function Alert({ variant, dismissible, children }: AlertProps) {
  return (
    <div className={cn(styles.alert, styles[variant])}>
      <span>{children}</span>
      {dismissible && (
        <button className={styles.dismiss}>Dismiss</button>
      )}
    </div>
  );
}
```


## Resources

- [Adding Styles](https://react.dev/learn#adding-styles) — React documentation on styling components

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*