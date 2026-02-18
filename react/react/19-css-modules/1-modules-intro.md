---
source_course: "react"
source_lesson: "react-css-modules-intro"
---

# CSS Modules

CSS Modules are CSS files where all class names are scoped locally by default.

## Benefits

- **No class name collisions**: Classes are automatically scoped
- **Explicit dependencies**: Import styles where you use them
- **Composition**: Extend and compose styles

## How It Works

1. Create a file with `.module.css` extension
2. Import it as an object
3. Use classes as object properties

## Code Examples

**CSS Module with composition**

```css
/* Button.module.css */
.container {
  padding: 20px;
  background-color: #f0f0f0;
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.primary {
  composes: button;
  background-color: #007bff;
  color: white;
}

.secondary {
  composes: button;
  background-color: #6c757d;
  color: white;
}
```

**Using CSS Modules in React**

```tsx
import styles from './Button.module.css';

function Button({ variant = 'primary', children }) {
  return (
    <div className={styles.container}>
      <button className={styles[variant]}>
        {children}
      </button>
    </div>
  );
}

// Multiple classes
function Card({ className }) {
  return (
    <div className={`${styles.card} ${className || ''}`}>
      {/* content */}
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*