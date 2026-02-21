---
source_course: "react-intermediate"
source_lesson: "react-css-modules-intro"
---

# Introduction to CSS Modules

## Introduction

One of the most frustrating problems in large CSS codebases is name collisions. Two developers independently create a `.card` class, and one silently overrides the other. CSS Modules solve this by automatically scoping class names to the component that imports them. Every class name becomes unique at build time, eliminating collisions entirely while letting you write plain, familiar CSS. No runtime overhead, no new syntax to learn — just CSS that works predictably at scale.

## Key Concepts

- **Scoped class names**: CSS Module class names are transformed at build time into unique identifiers (e.g., `.card` becomes `.Card_card_x7k3a`), preventing collisions across components.
- **`.module.css` extension**: Files must use the `.module.css` (or `.module.scss`) extension to be treated as CSS Modules by the bundler.
- **Import as object**: You import the CSS Module as a JavaScript object where keys are the original class names and values are the generated unique names.
- **Composition**: CSS Modules support a `composes` keyword that lets classes inherit styles from other classes, similar to Sass `@extend` but with explicit dependency tracking.

## Real World Context

Consider a design system used across 50 micro-frontends. Without CSS Modules, every team must coordinate class names to avoid collisions, or use BEM naming conventions that produce long, unwieldy names like `.design-system__card--highlighted`. With CSS Modules, every team writes simple, short class names like `.card` and `.highlighted`, and the build tool guarantees uniqueness. This simplifies the CSS, reduces bugs, and eliminates cross-team coordination for naming.

## Deep Dive

Create a CSS Module file:

```css
/* Button.module.css */
.container {
  display: flex;
  gap: 8px;
  padding: 16px;
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: background-color 0.2s;
}

.primary {
  composes: button;
  background-color: #2563eb;
  color: white;
}

.primary:hover {
  background-color: #1d4ed8;
}

.secondary {
  composes: button;
  background-color: #e5e7eb;
  color: #1f2937;
}

.secondary:hover {
  background-color: #d1d5db;
}
```

Import and use it in React:

```tsx
import styles from "./Button.module.css";

function ButtonGroup() {
  return (
    <div className={styles.container}>
      <button className={styles.primary}>Save</button>
      <button className={styles.secondary}>Cancel</button>
    </div>
  );
}
```

Combining multiple classes:

```tsx
import styles from "./Card.module.css";

function Card({ highlighted, className }: { highlighted?: boolean; className?: string }) {
  const classes = [
    styles.card,
    highlighted && styles.highlighted,
    className,
  ].filter(Boolean).join(" ");

  return <div className={classes}>Content</div>;
}
```

At build time, the class names are transformed:

```html
<!-- What you write -->
<div class="container">
  <button class="primary button">Save</button>
</div>

<!-- What the browser sees -->
<div class="Button_container_a3x2k">
  <button class="Button_primary_b7y9z Button_button_c1m4n">Save</button>
</div>
```

## Common Pitfalls

1. **Forgetting the `.module.css` extension** — A regular `.css` file is treated as global CSS. If you import it as an object, the keys will be undefined. Always use `.module.css` for scoped styles.
2. **Using hyphens in class names** — CSS class names with hyphens (like `.my-class`) require bracket notation: `styles["my-class"]`. Prefer camelCase (`.myClass`) to use dot notation: `styles.myClass`.

## Best Practices

1. **Use camelCase class names** — Write `.cardTitle` instead of `.card-title` so you can access it as `styles.cardTitle` instead of `styles["card-title"]`.
2. **Colocate CSS Modules with components** — Place `Button.module.css` next to `Button.tsx` in the same directory. This makes the style-component relationship obvious and keeps related code together.

## Summary

- CSS Modules automatically scope class names to prevent collisions, using the `.module.css` file extension.
- Import CSS Modules as JavaScript objects where keys are class names and values are generated unique identifiers.
- Use `composes` for style inheritance and camelCase class names for convenient dot notation access.

## Code Examples

**CSS Module usage with conditional class names**

```tsx
// Card.module.css (colocated with component)
// .card { border-radius: 8px; padding: 16px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
// .title { font-size: 18px; font-weight: bold; margin-bottom: 8px; }
// .highlighted { border: 2px solid #2563eb; }

import styles from "./Card.module.css";

function Card({ title, highlighted, children }: {
  title: string;
  highlighted?: boolean;
  children: React.ReactNode;
}) {
  return (
    <div className={`${styles.card} ${highlighted ? styles.highlighted : ""}`}>
      <h3 className={styles.title}>{title}</h3>
      {children}
    </div>
  );
}
```


## Resources

- [CSS Modules](https://react.dev/learn#adding-styles) — React docs on adding styles including CSS Modules

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*