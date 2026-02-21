---
source_course: "react-intermediate"
source_lesson: "react-css-modules-composition"
---

# CSS Modules Composition

## Introduction

One of CSS Modules' most powerful features is composition — the ability for one class to inherit styles from another. Unlike Sass `@extend`, which can produce unexpected CSS output with complex selectors, CSS Modules `composes` works by adding multiple class names to the element. It is explicit, predictable, and works across files, making it the foundation for building scalable style systems with CSS Modules.

## Key Concepts

- **`composes` keyword**: Lets a class inherit all styles from another class. The composed class gets both class names in the final HTML.
- **Same-file composition**: `composes: baseClass;` inherits from a class in the same file.
- **Cross-file composition**: `composes: baseClass from "./Base.module.css";` inherits from a class in another CSS Module file.
- **Values**: CSS Modules support `@value` declarations for sharing constants (colors, breakpoints) across files.

## Real World Context

A design system typically has base styles shared across components: all buttons share padding and font properties, all cards share border-radius and shadow, all inputs share height and border styles. With `composes`, you define these base styles once in a shared CSS Module file and compose them into variant-specific classes. When you update the base padding, every variant updates automatically.

## Deep Dive

**Same-file composition:**

```css
/* Button.module.css */
.base {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.primary {
  composes: base;
  background-color: #2563eb;
  color: white;
}

.secondary {
  composes: base;
  background-color: #f3f4f6;
  color: #374151;
}

.danger {
  composes: base;
  background-color: #dc2626;
  color: white;
}
```

When you use `styles.primary`, the generated HTML gets both class names:

```html
<button class="Button_primary_a1b2 Button_base_c3d4">Save</button>
```

**Cross-file composition:**

```css
/* shared/typography.module.css */
.heading {
  font-family: "Inter", sans-serif;
  font-weight: 700;
  line-height: 1.2;
}

.body {
  font-family: "Inter", sans-serif;
  font-weight: 400;
  line-height: 1.6;
}
```

```css
/* Card.module.css */
.title {
  composes: heading from "../shared/typography.module.css";
  font-size: 20px;
  margin-bottom: 8px;
}

.description {
  composes: body from "../shared/typography.module.css";
  font-size: 14px;
  color: #6b7280;
}
```

**Sharing values with `@value`:**

```css
/* shared/tokens.module.css */
@value blue: #2563eb;
@value red: #dc2626;
@value radius: 8px;
@value breakpoint-md: 768px;
```

```css
/* Button.module.css */
@value blue, radius from "../shared/tokens.module.css";

.button {
  background-color: blue;
  border-radius: radius;
}
```

## Common Pitfalls

1. **Composing from non-module CSS files** — `composes` only works with `.module.css` files. Trying to compose from a regular `.css` file will fail silently or produce errors.
2. **Order-dependent composition** — When a class composes from another, the composed styles come first. If both classes set the same property, the composing class wins. Be aware of this specificity behavior.

## Best Practices

1. **Create shared token files** — Put colors, spacing, and typography base classes in shared CSS Module files. Use `@value` for constants and `composes` for base class patterns.
2. **Keep composition shallow** — Avoid chaining `composes` through many levels (A composes B, B composes C, C composes D). One level of composition is usually enough and keeps the system easy to understand.

## Summary

- `composes` lets CSS Module classes inherit styles from other classes by adding multiple class names to the element.
- Cross-file composition enables shared base styles and design tokens across a component library.
- `@value` declarations share constants like colors and breakpoints between CSS Module files.

## Code Examples

**Cross-file composition for shared input base styles**

```css
/* shared/base.module.css */
.inputBase {
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  outline: none;
  transition: border-color 0.2s;
}

.inputBase:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.2);
}

/* TextInput.module.css */
.input {
  composes: inputBase from "../shared/base.module.css";
  width: 100%;
}

.error {
  composes: inputBase from "../shared/base.module.css";
  border-color: #dc2626;
}
```


## Resources

- [CSS Modules Composition](https://github.com/css-modules/css-modules/blob/master/docs/composition.md) — Official CSS Modules composition documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*