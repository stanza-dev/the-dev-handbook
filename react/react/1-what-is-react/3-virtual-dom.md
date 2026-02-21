---
source_course: "react"
source_lesson: "react-virtual-dom"
---

# The Virtual DOM

## Introduction

One of React's most important architectural decisions is the virtual DOM, a lightweight in-memory representation of the actual browser DOM. Understanding how the virtual DOM works will help you write more performant components and debug rendering issues. This lesson explains the reconciliation process that makes React fast.

## Key Concepts

- **Virtual DOM**: A plain JavaScript object tree that mirrors the structure of the real DOM. It is cheap to create and compare.
- **Reconciliation**: The algorithm React uses to diff the previous virtual DOM tree against the new one and compute the minimal set of DOM mutations.
- **Fiber Architecture**: React's internal engine (introduced in React 16) that breaks rendering work into small units and can pause, prioritize, or abort work as needed.
- **Batching**: React groups multiple state updates into a single re-render for better performance.

## Real World Context

Consider an e-commerce product page where the user adds an item to the cart, which updates the cart badge count in the header, the subtotal in the sidebar, and a confirmation toast. Without the virtual DOM, you would need to imperatively find each of those DOM nodes and update them individually. With React, you update the cart state once, and reconciliation ensures that only the three affected DOM nodes are touched, leaving the rest of the page untouched.

## Deep Dive

When a component's state changes, React performs the following steps:

1. **Re-render**: React calls the component function (and its children) to produce a new virtual DOM tree.
2. **Diff**: React compares the new tree to the previous tree node by node.
3. **Commit**: React applies only the differences to the actual browser DOM.

Here is a simplified visualization:

```tsx
// First render produces this virtual tree:
// <div>
//   <h1>Products</h1>
//   <span>Cart: 0</span>
// </div>

// After adding an item, the new tree is:
// <div>
//   <h1>Products</h1>      ← unchanged, skipped
//   <span>Cart: 1</span>   ← text changed, DOM updated
// </div>
```

React only updates the text content of the `<span>` because that is the only difference between the two trees. The `<div>` and `<h1>` are left untouched.

The Fiber architecture makes this even more sophisticated. Each node in the virtual DOM is represented as a "fiber" — a JavaScript object that holds information about the component, its state, and its place in the tree. React processes fibers in small chunks, yielding control back to the browser between chunks so that animations and user input remain responsive. This is called **concurrent rendering** and is enabled by default in React 18 and 19.

```tsx
function ProductPage() {
  const [cartCount, setCartCount] = useState(0);

  const handleAddToCart = () => {
    // React batches this update and only re-renders once
    setCartCount(prev => prev + 1);
  };

  return (
    <div>
      <h1>Products</h1>
      <span>Cart: {cartCount}</span>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}
```

In the example above, clicking the button triggers a state update. React re-invokes `ProductPage`, diffs the new output, and patches only the `<span>` text node.

## Common Pitfalls

1. **Thinking the virtual DOM makes everything fast automatically** — The virtual DOM reduces unnecessary DOM mutations, but if your component tree is very large and re-renders frequently, you still need to optimize with `React.memo`, `useMemo`, or the React Compiler.
2. **Confusing the virtual DOM with Shadow DOM** — Shadow DOM is a browser API for style encapsulation (used by Web Components). The virtual DOM is a React-specific concept for efficient updates. They are completely unrelated.

## Best Practices

1. **Keep component trees shallow** — Deeply nested trees take longer to diff. Extract sub-trees into separate components so React can bail out of unchanged branches early.
2. **Use stable keys in lists** — Keys are how React matches old and new elements during reconciliation. Unstable keys (like `Math.random()`) force React to destroy and recreate DOM nodes unnecessarily.

## Summary

- The virtual DOM is a lightweight JavaScript representation of the real DOM that React uses to compute efficient updates.
- Reconciliation diffs the old and new virtual DOM trees and applies only the minimal changes to the browser.
- React's Fiber architecture enables concurrent rendering, keeping the UI responsive even during large updates.

## Code Examples

**When cartCount changes, React only updates the span text node, not the entire DOM tree**

```tsx
import { useState } from 'react';

function ProductPage() {
  const [cartCount, setCartCount] = useState(0);

  const handleAddToCart = () => {
    setCartCount(prev => prev + 1);
  };

  return (
    <div>
      <h1>Products</h1>
      <span>Cart: {cartCount}</span>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}
```


## Resources

- [Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state) — Official guide on how React tracks component identity in the tree
- [React Fiber Architecture](https://react.dev/learn/render-and-commit) — Official explanation of the render and commit phases

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*