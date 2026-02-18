---
source_course: "react"
source_lesson: "react-introduction-to-react"
---

# Introduction to React

React is a JavaScript library for building user interfaces, created by Facebook (Meta). It lets you build complex UIs from small, isolated pieces of code called "components."

## Why React?

- **Component-Based**: Build encapsulated components that manage their own state
- **Declarative**: Describe what your UI should look like, React handles the DOM updates
- **Learn Once, Write Anywhere**: Works on web, mobile (React Native), and more

## Key Concepts

1. **Components**: Reusable UI building blocks
2. **JSX**: Syntax extension for writing HTML-like code in JavaScript
3. **Virtual DOM**: React's efficient diffing algorithm for updates
4. **One-way Data Flow**: Data flows down from parent to child components

## Code Examples

**Your first React component**

```tsx
// A simple React component
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Using the component
<Welcome name="Developer" />
```


## Resources

- [React Official Documentation](https://react.dev) — The official React documentation - start here

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*