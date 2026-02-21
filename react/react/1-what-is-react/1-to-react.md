---
source_course: "react"
source_lesson: "react-introduction-to-react"
---

# Introduction to React

## Introduction

React is the most widely adopted JavaScript library for building user interfaces, originally created by Meta (formerly Facebook) and now maintained by an extensive open-source community. Whether you are building a small interactive widget or a full-scale web application, React gives you a powerful mental model for composing UIs out of isolated, reusable pieces called components. In this lesson you will learn what React is, why it was created, and the core ideas that set it apart from other front-end tools.

## Key Concepts

- **Component-Based Architecture**: React applications are built from self-contained components that each manage their own markup, styles, and behavior.
- **Declarative Rendering**: You describe *what* the UI should look like for a given state, and React figures out *how* to update the browser DOM efficiently.
- **Unidirectional Data Flow**: Data flows downward from parent components to children through props, making the application easier to reason about and debug.
- **JSX**: A syntax extension that lets you write HTML-like markup directly inside JavaScript, which React transforms into function calls that produce UI elements.

## Real World Context

Imagine you are building a social media feed. Without React, you would manually query the DOM, create elements, attach event listeners, and track which parts of the page need updating when new posts arrive. With React, you describe a `Post` component once and render a list of them. When the data changes, React automatically reconciles the differences and updates only the parts of the DOM that actually changed. Teams at Meta, Airbnb, Netflix, and thousands of other companies rely on this model to ship complex UIs with confidence.

## Deep Dive

React was first released in 2013 to solve a specific problem at Facebook: keeping the UI in sync with rapidly changing data. The key insight was that re-rendering the entire UI conceptually (as if starting from scratch) and then efficiently diffing the result against the current DOM is simpler and less error-prone than imperatively mutating individual DOM nodes.

A minimal React application looks like this:

```tsx
import { createRoot } from 'react-dom/client';

function App() {
  return <h1>Hello, React!</h1>;
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

In the code above, `App` is a function component that returns JSX. The `createRoot` API from React 18+ mounts the component tree into a real DOM node. Every time state inside `App` (or its children) changes, React re-invokes the relevant components, builds a lightweight in-memory representation of the UI (the virtual DOM), compares it to the previous version, and applies the minimal set of DOM mutations required.

React 19 continues this philosophy while adding improvements like the React Compiler for automatic memoization and enhanced server component support. The library itself remains focused on the view layer, which means you combine it with other tools (React Router, TanStack Query, Next.js) for routing, data fetching, and server-side rendering.

## Common Pitfalls

1. **Treating React as a full framework** — React is a library focused on rendering UI. You still need separate solutions for routing, state management, and data fetching. Choosing a meta-framework like Next.js gives you these pieces out of the box.
2. **Mutating data directly** — React relies on detecting new object references to know when to re-render. If you mutate an existing object or array instead of creating a new one, the UI will not update.

## Best Practices

1. **Start with a framework** — The React team recommends starting new projects with Next.js or another React framework rather than a bare Vite setup, because frameworks handle routing, code-splitting, and server rendering for you.
2. **Think in components** — Break your UI into the smallest meaningful pieces. A good component does one thing well and can be reused or tested in isolation.

## Summary

- React is a JavaScript library for building UIs through composable, declarative components.
- It uses a virtual DOM diffing strategy to efficiently update only the parts of the page that changed.
- Data flows one way (parent to child) via props, which makes applications predictable and easy to debug.

## Code Examples

**A minimal React application with a function component and typed props**

```tsx
import { createRoot } from 'react-dom/client';

function Welcome({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

const root = createRoot(document.getElementById('root')!);
root.render(<Welcome name="Developer" />);
```


## Resources

- [React Official Documentation](https://react.dev) — The official React documentation - start here
- [Thinking in React](https://react.dev/learn/thinking-in-react) — Official guide on the React mental model

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*